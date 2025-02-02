Проектная работа по теме "Переход сети ДЦ со статического VXLAN на EVPN".

Цель:

 - Настроить стаический L2\L3 VXLAN на коммутаторах, проверить их связность.
 - Перенастроить на EVPN+L2\L3 VNI и проверить их связность.


Описание:
Оборудование для лаборатории - Arista SW (4.29.2F) Для underlay будет использован eBGP.

Сети для loopback'ов:
SPINE1 - 172.31.0.1
SPINE2 - 172.31.0.2

LEAF1 - 172.16.0.1
LEAF2 - 172.16.0.2
LEAF3 - 172.16.0.3

Сети для связи коммутаторов указаны на схеме, т.к., и они почти не упоминаются.

Схема:

![CLOS](CLOS02.png)


**eBGP.**

Настройка eBGP с помощью peer group и префикс листа, чтобы редистрибьютить loopback'и.

```html

hostname SPINE1
!
spanning-tree mode mstp
!
interface Ethernet1
   no switchport
   ip address 10.1.0.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 10.1.0.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 10.1.0.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 172.31.0.1/32
!
interface Management1
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 172.31.0.0/16 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65001
   maximum-paths 3 ecmp 3
   neighbor 10.1.0.1 remote-as 65011
   neighbor 10.1.0.3 remote-as 65031
   neighbor 10.1.0.5 remote-as 65021
   redistribute connected route-map LOOPBACKS
!
end

```

```html

hostname SPINE2
!
spanning-tree mode mstp
!
interface Ethernet1
!
interface Ethernet2
   no switchport
   ip address 10.2.0.5/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 10.2.0.3/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
   no switchport
   ip address 10.2.0.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback1
   ip address 172.31.0.2/32
!
interface Management1
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 172.31.0.0/16 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65001
   maximum-paths 3 ecmp 3
   neighbor 10.2.0.1 remote-as 65011
   neighbor 10.2.0.2 remote-as 65021
   neighbor 10.2.0.4 remote-as 65031
   redistribute connected route-map LOOPBACKS
!
end

```

Далее, лифы.

```html
hostname LEAF1
!
interface Ethernet1
   no switchport
   ip address 10.1.0.1/31

interface Ethernet4
   no switchport
   ip address 10.2.0.1/31

interface Loopback1
   ip address 172.16.0.1/32
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 172.16.0.0/16 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65011
   maximum-paths 2 ecmp 2
   neighbor 10.1.0.0 remote-as 65001
   neighbor 10.2.0.0 remote-as 65001
   redistribute connected route-map LOOPBACKS
!
end

```

```html


hostname LEAF2
!
interface Ethernet2
   no switchport
   ip address 10.1.0.5/31
!
interface Ethernet3
   no switchport
   ip address 10.2.0.2/31
   interface Loopback1
   ip address 172.16.0.2/32
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 172.16.0.0/16 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65021
   maximum-paths 2 ecmp 2
   neighbor 10.1.0.4 remote-as 65001
   neighbor 10.2.0.3 remote-as 65001
   redistribute connected route-map LOOPBACKS
!
end

```

```html


hostname LEAF3
!

interface Ethernet2
   no switchport
   ip address 10.2.0.4/31

!
interface Ethernet3
   no switchport
   ip address 10.1.0.3/31
  interface Loopback1
   ip address 172.16.0.3/32
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 172.16.0.0/16 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65031
   maximum-paths 2 ecmp 2
   neighbor 10.1.0.2 remote-as 65001
   neighbor 10.2.0.5 remote-as 65001
   redistribute connected route-map LOOPBACKS
!
end

```

Проверка:

```html

LEAF1#ping 172.16.0.2 source lo1
PING 172.16.0.2 (172.16.0.2) from 172.16.0.1 : 72(100) bytes of data.
80 bytes from 172.16.0.2: icmp_seq=1 ttl=63 time=11.3 ms
80 bytes from 172.16.0.2: icmp_seq=2 ttl=63 time=7.03 ms
80 bytes from 172.16.0.2: icmp_seq=3 ttl=63 time=6.03 ms
80 bytes from 172.16.0.2: icmp_seq=4 ttl=63 time=5.64 ms
80 bytes from 172.16.0.2: icmp_seq=5 ttl=63 time=5.40 ms

--- 172.16.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 5.401/7.088/11.335/2.196 ms, ipg/ewma 10.961/9.103 ms
LEAF1#ping 172.16.0.3 source lo1
PING 172.16.0.3 (172.16.0.3) from 172.16.0.1 : 72(100) bytes of data.
80 bytes from 172.16.0.3: icmp_seq=1 ttl=63 time=9.56 ms
80 bytes from 172.16.0.3: icmp_seq=2 ttl=63 time=5.71 ms
80 bytes from 172.16.0.3: icmp_seq=3 ttl=63 time=5.45 ms
80 bytes from 172.16.0.3: icmp_seq=4 ttl=63 time=5.71 ms
80 bytes from 172.16.0.3: icmp_seq=5 ttl=63 time=5.33 ms

--- 172.16.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 37ms
rtt min/avg/max/mdev = 5.332/6.357/9.568/1.612 ms, ipg/ewma 9.298/7.901 ms

```

**Static VXLAN**

По сути, надо настроить один интерфейс VXLAN на трех LEAF'х:

```html

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 88 vni 10088
   vxlan vlan 99 vni 10099
   vxlan flood vtep 172.16.0.1 172.16.0.2

```

```html
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 99 vni 10099
   vxlan flood vtep 172.16.0.1 172.16.0.3

```

```html

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 88 vni 10088
   vxlan vlan 99 vni 10099
   vxlan flood vtep 172.16.0.2 172.16.0.3

```

Для проверки сделаем пару хостов, пока что, не указывая на них шлюз по умалчанию:

На LEAF1 - 192.168.99.101/24
На LEAF2 - 192.168.99.102/24

```html

VPCS> ip 192.168.99.100/24
Checking for duplicate address...
VPCS : 192.168.99.100 255.255.255.0

VPCS> ping 192.168.99.101

84 bytes from 192.168.99.101 icmp_seq=1 ttl=64 time=17.805 ms
84 bytes from 192.168.99.101 icmp_seq=2 ttl=64 time=15.256 ms
84 bytes from 192.168.99.101 icmp_seq=3 ttl=64 time=14.718 ms
84 bytes from 192.168.99.101 icmp_seq=4 ttl=64 time=14.486 ms
84 bytes from 192.168.99.101 icmp_seq=5 ttl=64 time=14.761 ms

```

**VXLAN EVPN**

Чтобы не было простоя связи, команды можно вводить последовательно, тогда будет минимум потерь пакетов.

