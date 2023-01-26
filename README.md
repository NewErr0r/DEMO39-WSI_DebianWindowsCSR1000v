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
    </ul>
</ul>
<ul>
