## 3.6. Компьютерные сети, лекция 2
---
1. >Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?

linux:
`ifconfig`, 
`ip a`

windows:
`ipconfig /all`

macos:
`ifconfig`, 
`networksetup -listnetworkserviceorder`
```
[vainoord@vnrd-mypc:devops-netology]$ ifconfig
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
	options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
	inet 127.0.0.1 netmask 0xff000000 
	inet6 ::1 prefixlen 128 
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1 
	nd6 options=201<PERFORMNUD,DAD>
gif0: flags=8010<POINTOPOINT,MULTICAST> mtu 1280
stf0: flags=0<> mtu 1280
ap1: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
	options=400<CHANNEL_IO>
	ether a6:83:e7:d2:27:fe 
	media: autoselect
	status: inactive
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=6463<RXCSUM,TXCSUM,TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
	ether a4:83:e7:d2:27:fe 
	inet6 fe80::869:8bee:f7c0:6e21%en0 prefixlen 64 secured scopeid 0x6 
	inet 172.19.220.11 netmask 0xffffff80 broadcast 172.19.220.127
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
awdl0: flags=8943<UP,BROADCAST,RUNNING,PROMISC,SIMPLEX,MULTICAST> mtu 1500
	options=400<CHANNEL_IO>
	ether 9a:89:ee:8a:69:d0 
	inet6 fe80::9889:eeff:fe8a:69d0%awdl0 prefixlen 64 scopeid 0x7 
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
llw0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=400<CHANNEL_IO>
	ether 9a:89:ee:8a:69:d0 
	inet6 fe80::9889:eeff:fe8a:69d0%llw0 prefixlen 64 scopeid 0x8 
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
``` 

```
[vainoord@vnrd-mypc:~]$ networksetup -listnetworkserviceorder
An asterisk (*) denotes that a network service is disabled.
(1) USB 10/100 LAN
(Hardware Port: USB 10/100 LAN, Device: en6)

(2) Wi-Fi
(Hardware Port: Wi-Fi, Device: en0)

(3) Thunderbolt Bridge
(Hardware Port: Thunderbolt Bridge, Device: bridge0)

(4) VM_bridge
(Hardware Port: VM_bridge, Device: bridge1)
```
---
2. >Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?

CDP и LLDP. Первый - проприетарный протокол Cisco, используется только на их устройствах. Второй - open source протокол для разных вендоров.

В Linux есть пакет lldpd.
Команды:
`lldpcli`, `lldpctl`

--- 
3. > Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.

VLAN технология.
В Linux существует одноименный пакет vlan.

Пример настройки в `/etc/network/interfaces`:
```
auto eth0
iface eth0 inet static
address 192.168.1.10
netmask 255.255.255.0
gateway 192.168.1.1

auto eth0.500
iface eth0.500 inet static
address 172.19.120.29
netmask 255.255.240.0
vlan_raw_device eth0
```
С такой конфигурацией на сетевой порт eth0 будет прописан vlan 500. Нетегированный трафик будет приходить в подсеть 192.168.1.0/24. Тегированный трафик с тегом 500 - в подсеть 172.19.120.16/28.

 

---
4. >Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.

LACP (Link Aggregation Control Protocol) - cпецификация, позволяющая объединять несколько физических каналов Ethernet в наших сетевых устройствах для формирования единого логического канала и включения балансировки нагрузки в наших интерфейсах.
В Cisco есть проприетарный протокол PAgP (Port Aggregation Protocol).

В linux для этого используется Bonding – объединение сетевых интерфейсов по типу агрегации, необходимо для увеличения пропускной способности и создания отказоустойчивости на сегменте сети.

Опции балансировки:
* Mode 0: Balance Round-Robin (balance-rr) - передача пакетов в последовательном порядке начиная с первого доступного slave порта. Этот режим обеспечивает балансировку нагрузки и отказоустойчивость. Используется по-умолчанию.
* Mode 1: Active-Backup - активен только один порт из пары. Другой в ожидании и становится активным тогда и только тогда, когда активный первый порт выходит из строя. Обеспечивает отказоустойчивость.
* Mode 2: Balance-xor - передача на основе выбранной политики хэширования передачи. Политика по умолчанию [(MAC-адрес источника XOR с MAC-адресом назначения) % количество портов]. Этот режим обеспечивает балансировку нагрузки и отказоустойчивость.
* Mode 3: Broadcast - с такой опцией передача трафика идет во все объединенные интерфейсы. Этот режим обеспечивает отказоустойчивость.
* Mode 4: 802.3ad - LACP. Динамическое объединение портов. Обеспечение отказоустойчивости и увеличение пропускной способности трафика между узлами входящего и исходящего трафика.
* Mode 5 – Adaptive transmit load balancing (balance-tlb) - адаптивная балансировка нагрузки при передаче: объединение каналов, не требующее специальной поддержки коммутатора. Исходящий трафик распределяется в соответствии с текущей нагрузкой (вычисляемой относительно скорости) на каждом ведомом устройстве. Входящий трафик принимается текущим ведомым устройством. Если принимающее ведомое устройство выходит из строя, другое ведомое устройство принимает MAC-адрес отказавшего принимающего ведомого устройства.
* Mode 6: Adaptive load balancing (balance-alb) - адаптивная балансировка нагрузки: включает balance-tlb плюс балансировку нагрузки на прием (rlb) для трафика IPV4 и не требует какой-либо специальной поддержки коммутатора. Балансировка нагрузки на прием достигается путем согласования ARP.

Пример конфигурации - агрегация ethernet порта и wlan порта на ноутбуке в mode 1:
```
# Define slaves - eth0 
auto eth0
iface eth0 inet manual
    bond-master bond0
    bond-primary eth0
    bond-mode active-backup

# Define slaves - wlan0
auto wlan0
iface wlan0 inet manual
    wpa-conf /etc/network/wpa.conf
    bond-master bond0
    bond-primary eth0
    bond-mode active-backup

# Define master - bond0
auto bond0
iface bond0 inet dhcp
    bond-slaves none
    bond-primary eth0
    bond-mode active-backup
    bond-miimon 100
```


---
5. >Сколько IP адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.


В сети с маской /29 - 8 адресов:
network address - 1
hosts - 6
broadcast address - 1 

В сети с маской /24 - 256 адресов. Такую сеть можно разделить на 32 подсети с маской /29.
В случае с сетью 10.10.10.0/24 будут следующие подсети с маской /29:

10.10.10.0/29 (10.10.10.1-10.10.10.6)
10.10.10.8/29 (10.10.10.9-10.10.10.14)
10.10.10.16/29 (10.10.10.17-10.10.10.22)
10.10.10.24/29 (10.10.10.25-10.10.10.30)
10.10.10.32/29 (10.10.10.33-10.10.10.38)
...
10.10.10.240/29 (10.10.10.241-10.10.10.246)
10.10.10.248/29 (10.10.10.249-10.10.10.254)


---
6. >Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.

Частные IPv4 адреса можно задействовать из диапазона IP 100.64.0.0-100.127.255.255 (Carrier-Grade NAT). Для 40-50 хостов необходимо взять маску подсети /26 - в такой подсети может быть 62 хоста.


---
7. >Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?
В linux:
просмотр ARP таблицы - `arp -n`, `arp -a`, `ip neigh`:
```
vagrant@dev-vm:~$ arp -n
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.2.2                 ether   52:54:00:12:35:02   C                     eth0
10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0

vagrant@dev-vm:~$ ip neigh show
10.0.2.2 dev eth0 lladdr 52:54:00:12:35:02 DELAY
10.0.2.3 dev eth0 lladdr 52:54:00:12:35:03 STALE
```
Для очистки ARP кеш полностью -`ip -s -s neigh flush all`
Удалить одну запись по IP из ARP таблицы - `arp -d 192.168.0.5`

В windows:
просмотр ARP таблицы - `arp -a`.
Для очистки ARP кеш полностью - `arp -d`
Удалить одну запись по IP из ARP таблицы - `arp -d 192.168.0.5`

---


