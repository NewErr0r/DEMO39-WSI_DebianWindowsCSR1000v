<h1>DEMO2023</h1>

<h1>Образец задания</h1>

<p>Образец задания для демонстрационного экзамена по комплекту оценочной документации.</p>

<h1>Описание задания</h1>

![Image alt](https://github.com/NewErr0r/DEMO39-WSI_DebianWindowsCSR1000v/blob/main/images/topology.png?raw=true)

<h1>Таблица 1. Характеристики ВМ</h1>

![Image alt](https://github.com/NewErr0r/DEMO39-WSI_DebianWindowsCSR1000v/blob/main/images/Specifications_VM's.png?raw=true)

<h2>Виртуальные машины и коммутация</h2>

<p>Необходимо выполнить создание и базовую конфигурацию виртуальных машин.</p>

<ul>
    <li>На основе предоставленных ВМ или шаблонов ВМ создайте отсутствующие виртуальные машины в соответствии со схемой</li>
    <ul>
      <li>Характеристики ВМ установите в соответствии с Таблицей 1;</li>
      <li>Коммутацию (если таковая не выполнена) выполните в соответствии со схемой сети.</li>
    </ul>
    <li>Имена хостов в созданных ВМ должны быть установлены в соответствии со схемой.</li>
    <li>Адресация должна быть выполнена в соответствии с Таблицей 1;</li>
    <li>Обеспечьте ВМ дополнительными дисками, если таковое необходимо в соответствии с Таблицей 1;</li>
</ul>

<h4>ISP</h4>

<pre>
hostnamectl set-hostname ISP
</pre>
<pre>
vi /etc/network/interfaces      # enp0s3 NAT-адаптер ( доступ в Интернет )
    
    auto enp0s3 
    iface enp0s3 inet dhcp
    
    auto enp0s8
    iface enp0s8 inet static
    address 3.3.3.1/24
    
    auto enp0s9
    iface enp0s9 inet static
        address 4.4.4.1/24
        dns-search demo.wsr
    
    auto enp0s10
    iface enp0s10 inet static
    address 5.5.5.1/24   
</pre>
<pre>
systemctl restart networking
</pre>

<h4>RTR-L</h4>

<pre>
hostname RTR-L
</pre>
<pre>
interface Gi1
ip address 4.4.4.100 255.255.255.0
no shutdown
exit<br>
interface Gi2
ip address 192.168.100.254 255.255.255.0
no shutdown
exit
</pre>


<h4>RTR-R</h4>

<pre>
hostname RTR-R
</pre>
<pre>
interface Gi1
ip address 5.5.5.100 255.255.255.0
no shutdown
exit<br>
interface Gi2
ip address 172.16.100.254 255.255.255.0
no shutdown
exit
</pre>

<h4>SRV</h4>

<pre>
Rename-Computer -NewName SRV
Restart-Computer
</pre>

![Image alt](https://github.com/NewErr0r/DEMO39-WSI_DebianWindowsCSR1000v/blob/main/images/SRV_addressing.png?raw=true)


<h4>WEB-L</h4>

<pre>
hostnamectl set-hostname WEB-L
</pre>
<pre>
vi /etc/network/interfaces
    
    auto enp0s3 
    iface enp0s3 inet static
        address 192.168.100.200/24
        gateway 192.168.100.254
        dns-nameservers 192.168.100.200
        dns-search int.demo.wsr
</pre>
<pre>
systemctl restart networking
</pre>

<h4>WEB-R</h4>

<pre>
hostnamectl set-hostname WEB-R
</pre>
<pre>
vi /etc/network/interfaces
    
    auto enp0s3 
    iface enp0s3 inet static
        address 172.16.100.200/24
        gateway 172.16.100.254
        dns-nameservers 192.168.100.200
        dns-search int.demo.wsr
</pre>
<pre>
systemctl restart networking
</pre>


<h4>CLI</h4>

<pre>
Rename-Computer -NewName CLI
Restart-Computer
</pre>

![Image alt](https://github.com/NewErr0r/DEMO39-WSI_DebianWindowsCSR1000v/blob/main/images/CLI_addressing.png?raw=true)

<h2>Сетевая связность</h2>

<p>В рамках данного модуля требуется обеспечить сетевую связность между регионами работы приложения, а также обеспечить выход ВМ в имитируемую сеть “Интернет”.</p>

<ul>
    <li>Сети, подключенные к ISP, считаются внешними:</li>
    <ul>
      <li>Запрещено прямое попадание трафика из внутренних сетей во внешние и наоборот;</li>
        <h4>ISP</h4>
        <pre>echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf<br> sysctl -p</pre>
        <pre>apt install -y firewalld<br>systemctl enable --now firewalld</pre>
        <pre>firewall-cmd --permanent --zone=trusted --add-interface=enp0s{8,9,10}<br>firewall-cmd --permanent --add-masquerade<br>firewall-cmd --reload</pre>
        <h4>RTR-L</h4>
        <pre>ip route 0.0.0.0 0.0.0.0 4.4.4.1</pre>
        <h4>RTR-R</h4>
        <pre>ip route 0.0.0.0 0.0.0.0 5.5.5.1</pre>
    </ul>
    <li>Платформы контроля трафика, установленные на границах регионов, должны выполнять трансляцию трафика, идущего из соответствующих внутренних сетей во внешние сети стенда и в сеть Интернет.</li>
    <ul>
      <li>Трансляция исходящих адресов производится в адрес платформы, расположенный во внешней сети.</li>
        <h4>RTR-L</h4>
        <pre>access-list 1 permit 192.168.100.0 0.0.0.255<br>ip nat inside source list 1 interface Gi1 overload</pre>
        <pre>interface Gi1<br>ip nat outside<br>exit<br>interface Gi2<br>ip nat inside<br>exit</pre>
        <h4>RTR-R</h4>
        <pre>access-list 1 permit 172.16.100.0 0.0.0.255<br>ip nat inside source list 1 interface Gi1 overload</pre>
        <pre>interface Gi1<br>ip nat outside<br>exit<br>interface Gi2<br>ip nat inside<br>exit</pre>
    </ul>
</ul>

<ul>
    <li>Между платформами должен быть установлен защищенный туннель, позволяющий осуществлять связь между регионами применением внутренних адресов.</li>
    <ul>
    <li>Трафик, проходящий по данному туннелю, должен быть защищен:</li>
        <ul>
        <li>Платформа ISP не должна иметь возможности просматривать содержимое пакетов, идущих из одной внутренней сети в другую.</li>
        </ul>
    <li>Туннель должен позволять защищенное взаимодействие между платформами управления трафиком по их внутренним адресам</li>
        <ul>
        <li>Взаимодействие по внешним адресам должно происходит без применения туннеля и шифрования</li>
        </ul>
    <li>Трафик, идущий по туннелю между регионами по внутренним адресам, не должен транслироваться.</li>
    <h4>RTR-L</h4>
    <pre>interface Tunnel1<br>ip address 10.20.30.1 255.255.255.252<br>tunnel mode gre ip<br>tunnel source 4.4.4.100<br>tunnel destination 5.5.5.100</pre>
    <pre>router bgp 65001<br>neighbor 10.20.30.2 remote-as 65002<br>network 10.20.30.0 mask 255.255.255.252<br>network 192.168.100.0 mask 255.255.255.0</pre>
    <h4>RTR-R</h4>
    <pre>interface Tunnel1<br>ip address 10.20.30.2 255.255.255.252<br>tunnel mode gre ip<br>tunnel source 5.5.5.100<br>tunnel destination 4.4.4.100</pre>
    <pre>router bgp 65002<br>neighbor 10.20.30.1 remote-as 65001<br>network 10.20.30.0 mask 255.255.255.252<br>network 172.16.100.0 mask 255.255.255.0</pre>
    <h4>RTR-L</h4>
    <pre>crypto isakmp policy 1<br>encryption aes<br>authentication pre-share<br>hash sha256<br>group14<br>exit<br><br>crypto isakmp key P@ssw0rd address 5.5.5.100<br>crypto isakmp nat keepalive 5<br><br>crypto ipsec transform-set TSET  esp-aes 256 esp-sha256-hmac<br>mode tunnel<br>exit<br><br>crypto ipsec profile VTI<br>set transform-set TSET<br>exit</pre>
    <pre>interface Tunnel1<br>tunnel mode ipsec ipv4<br>tunnel protection ipsec profile VTI</pre>
    <h4>RTR-R</h4>
<pre>crypto isakmp policy 1<br>encryption aes<br>authentication pre-share<br>hash sha256<br>group14<br>exit<br><br>crypto isakmp key P@ssw0rd address 4.4.4.100<br>crypto isakmp nat keepalive 5<br><br>crypto ipsec transform-set TSET  esp-aes 256 esp-sha256-hmac<br>mode tunnel<br>exit<br><br>crypto ipsec profile VTI<br>set transform-set TSET<br>exit</pre>
    <pre>interface Tunnel1<br>tunnel mode ipsec ipv4<br>tunnel protection ipsec profile VTI</pre>
    </ul>
</ul>
<ul>
    <li>Платформа управления трафиком RTR-L выполняет контроль входящего трафика согласно следующим правилам:</li>
    <ul>
        <li>Разрешаются подключения к портам DNS, HTTP и HTTPS для всех клиентов;</li>
        <ul>
            <li>Порты необходимо для работы настраиваемых служб</li>
        </ul>
        <li>Разрешается работа выбранного протокола организации защищенной связи;</li>
        <ul>
            <li>Разрешение портов должно быть выполнено по принципу “необходимо и достаточно”</li>
        </ul>
        <li>Разрешается работа протоколов ICMP;</li>
        <li>Разрешается работа протокола SSH;</li>
        <li>Прочие подключения запрещены;</li>
        <li>Для обращений в платформам со стороны хостов, находящихся внутри регионов, ограничений быть не должно;</li>
        <h4>RTR-L</h4>
        <pre>ip access-list extended LEFT<br><br>permit tcp any any established<br>permit udp host 4.4.4.100 eq 53 any<br>permit udp host 5.5.5.1 eq 123 any<br>permit tcp any host 4.4.4.100 eq 80 <br>permit tcp any host 4.4.4.100 eq 443 <br>permit tcp any host 4.4.4.100 eq 2222 <br><br>permit udp host 5.5.5.100 host 4.4.4.100 eq 500<br>permit esp any any<br>permit icmp any any<br>exit</pre>
        <pre>interface Gi1<br>ip access-group LEFT in</pre>
    </ul>
    <li>Платформа управления трафиком RTR-R выполняет контроль входящего трафика согласно следующим правилам:</li>
    <ul>
        <li>Разрешаются подключения к портам HTTP и HTTPS для всех клиентов;</li>
        <ul>
            <li>Порты необходимо для работы настраиваемых служб</li>
        </ul>
        <li>Разрешается работа выбранного протокола организации защищенной связи;</li>
        <ul>
            <li>Разрешение портов должно быть выполнено по принципу “необходимо и достаточно”</li>
        </ul>
        <li>Разрешается работа протоколов ICMP;</li>
        <li>Разрешается работа протокола SSH;</li>
        <li>Прочие подключения запрещены;</li>
        <li>Для обращений в платформам со стороны хостов, находящихся внутри регионов, ограничений быть не должно;</li>
        <h4>RTR-R</h4>
        <pre>ip access-list extended RIGHT<br><br>permit tcp any any established<br>permit tcp any host 5.5.5.100 eq 80 <br>permit tcp any host 5.5.5.100 eq 443 <br>permit tcp any host 5.5.5.100 eq 2244 <br><br>permit udp host 4.4.4.100 host 5.5.5.100 eq 500<br>permit esp any any<br>permit icmp any any<br>exit</pre> 
        <pre>interface Gi1<br>ip access-group RIGHT in</pre>
    </ul>
</ul>
<ul>
    <li>Обеспечьте настройку служб SSH региона Left:</li>
    <ul>
        <li>Подключения со стороны внешних сетей по протоколу к платформе управления трафиком RTR-L на порт 2222 должны быть перенаправлены на ВМ Web-L;</li>
        <li>Подключения со стороны внешних сетей по протоколу к платформе управления трафиком RTR-R на порт 2244 должны быть перенаправлены на ВМ Web-R;</li>
        <h4>RTR-L</h4>
        <pre>ip nat inside source static tcp 192.168.100.100 22 4.4.4.100 2222</pre>
        <h4>RTR-R</h4>
        <pre>ip nat inside source static tcp 172.16.100.100 22 5.5.5.100 2244</pre>
        <h4>WEB-L</h4>
        <pre>vi /etc/ssh/sshd_config<br>...<br>PermitRootLogin yes<br>...</pre>
        <pre>systemctl restart sshd</pre>
        <h4>WEB-R</h4>
        <pre>vi /etc/ssh/sshd_config<br>...<br>PermitRootLogin yes<br>...</pre>
        <pre>systemctl restart sshd</pre>
    </ul>
</ul>
<h2>Инфраструктурные службы</h2>

<p>В рамках данного модуля необходимо настроить основные инфраструктурные службы и настроить представленные ВМ на применение этих служб для всех основных функций.</p>

<h1>Таблица 2. DNS-записи зон</h1>

![Image alt](https://github.com/NewErr0r/DEMO39-WSI_DebianWindowsCSR1000v/blob/main/images/table_dns.png?raw=true)

<ul>
    <li>Выполните настройку первого уровня DNS-системы стенда:</li>
    <ul>
        <li>Используется ВМ ISP;</li>
        <li>Обслуживается зона demo.wsr.</li>
        <ul>
            <li>Наполнение зоны должно быть реализовано в соответствии с Таблицей 2;</li>
        </ul>
        <li>Сервер делегирует зону int.demo.wsr на SRV;</li>
        <ul>
            <li>Поскольку SRV находится во внутренней сети западного региона, делегирование происходит на внешний адрес маршрутизатора данного региона.</li>
            <li>Маршрутизатор региона должен транслировать соответствующие порты DNS-службы в порты сервера SRV.</li>
        </ul>
        <li>Внешний клиент CLI должен использовать DNS-службу, развернутую на ISP, по умолчанию;</li>
        <h4>ISP</h4>
        <pre>apt install bind9 -y</pre>
        <pre>mkdir /opt/dns<br>cp /etc/bind/db.local /opt/dns/demo.db<br>chown -R bind:bind /opt/dns</pre>
        <pre>vi /etc/apparmor.d/usr.sbin.named<br>...<br>/opt/dns/** rw,<br>...</pre>
        <pre>systemctl restart apparmor.service</pre>
        <pre>vi /etc/bind/named.conf.options<br>options {
            directory "/var/cache/bind";
            dnssec-validation no;
            allow-query { any; };
            listen-on-v6 { any; };
        };</pre>
        <pre>vi /etc/bind/named.conf.default-zones<br>zone "demo.wsr" {
            type master;
            allow-transfer { any; };
            file "/opt/dns/demo.db";
        };</pre>
        <pre>vi /opt/dns/demo.db</pre>
        
![Image alt](https://github.com/NewErr0r/DEMO39-WSI_DebianWindowsCSR1000v/blob/main/images/demo.db.png?raw=true)

</pre>
<pre>systemctl restatr bind9</pre>
    <h4>RTR-L</h4>
    <pre>ip nat inside source static tcp 192.168.100.200 53 4.4.4.100 53<br>ip nat inside source static udp 192.168.100.200 53 4.4.4.100 53</pre>
    </ul>
</ul>
   <ul>
        <li>Выполните настройку второго уровня DNS-системы стенда;</li>
        <ul>
            <li>Используется ВМ SRV;</li>
            <li>Обслуживается зона int.demo.wsr;</li>
            <ul>
                <li>Наполнение зоны должно быть реализовано в соответствии с Таблицей 2;</li>
            </ul>
            <li>Обслуживаются обратные зоны для внутренних адресов регионов</li>
            <ul>
                <li>Имена для разрешения обратных записей следует брать из Таблицы 2;</li>
            </ul>
            <li>Сервер принимает рекурсивные запросы, исходящие от адресов внутренних регионов;</li>
            <ul>
                <li>Обслуживание клиентов(внешних и внутренних), обращающихся к зоне int.demo.wsr, должно производиться без каких-либо ограничений по адресу источника;</li>
            </ul>
            <li>Внутренние хосты регионов (равно как и платформы управления трафиком) должны использовать данную DNS-службу для разрешения всех запросов имен;</li>
            <h4>SRV</h4>
            <pre>Install-WindowsFeature -Name DNS -IncludeManagementTools</pre>
            <pre>Add-DnsServerPrimaryZone -Name "int.demo.wsr" -ZoneFile "int.demo.wsr.dns"</pre>
            <pre>Add-DnsServerPrimaryZone -NetworkId 192.168.100.0/24 -ZoneFile "int.demo.wsr.dns"<br>Add-DnsServerPrimaryZone -NetworkId 172.16.100.0/24 -ZoneFile "int.demo.wsr.dns"</pre>
            <pre>Add-DnsServerResourceRecordA -Name "web-l" -ZoneName "int.demo.wsr" -AllowUpdateAny -IPv4Address "192.168.100.100" -CreatePtr <br>Add-DnsServerResourceRecordA -Name "web-r" -ZoneName "int.demo.wsr" -AllowUpdateAny -IPv4Address "172.16.100.100" -CreatePtr <br>Add-DnsServerResourceRecordA -Name "srv" -ZoneName "int.demo.wsr" -AllowUpdateAny -IPv4Address "192.168.100.200" -CreatePtr <br>Add-DnsServerResourceRecordA -Name "rtr-l" -ZoneName "int.demo.wsr" -AllowUpdateAny -IPv4Address "192.168.100.254" -CreatePtr <br>Add-DnsServerResourceRecordA -Name "rtr-r" -ZoneName "int.demo.wsr" -AllowUpdateAny -IPv4Address "172.16.100.254" -CreatePtr</pre>
            <pre>Add-DnsServerResourceRecordCName -Name "ntp" -HostNameAlias "srv.int.demo.wsr" -ZoneName "int.demo.wsr"<br>Add-DnsServerResourceRecordCName -Name "dns" -HostNameAlias "srv.int.demo.wsr" -ZoneName "int.demo.wsr"</pre>
            
            
![Image alt](https://github.com/NewErr0r/DEMO39-WSI_DebianWindowsCSR1000v/blob/main/images/int.demo.wsr.db.png?raw=true)
            
<h4>RTR-L</h4>
<pre>ip name-server 192.168.100.200<br>ip domain name int.demo.wsr</pre>
<h4>RTR-R</h4>
<pre>ip name-server 192.168.100.200<br>ip domain name int.demo.wsr</pre>
            
</ul></ul>
        
