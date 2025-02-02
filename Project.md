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

Чтобы сдеать статическую маршрутизацию, добавим виртуальные адреса на интерфейсы VLAN, на LEAF 1/2.

```html
interface Vlan88
   ip address virtual 192.168.88.1/24
!
interface Vlan99
   ip address virtual 192.168.99.1/24
```

И проверяем связность, добавив хост в 88 VLAN.


```html

VPCS> sh ip  

NAME        : VPCS[1]
IP/MASK     : 192.168.88.101/24
GATEWAY     : 192.168.88.1
DNS         : 
MAC         : 00:50:79:66:68:0c
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.99.101

84 bytes from 192.168.99.101 icmp_seq=1 ttl=62 time=285.674 ms
84 bytes from 192.168.99.101 icmp_seq=2 ttl=62 time=18.032 ms
84 bytes from 192.168.99.101 icmp_seq=3 ttl=62 time=15.934 ms
84 bytes from 192.168.99.101 icmp_seq=4 ttl=62 time=17.469 ms
84 bytes from 192.168.99.101 icmp_seq=5 ttl=62 time=16.899 ms

VPCS> 

```

**VXLAN EVPN**

Чтобы не было простоя связи, команды можно вводить последовательно, тогда будет минимум потерь пакетов.

Настроим eBGP для EVPN, последовательно для LEAF 1,2,3.

```html

router bgp 65011
   maximum-paths 2 ecmp 2
   neighbor OVERLAY peer group
   neighbor OVERLAY update-source Loopback1
   neighbor OVERLAY ebgp-multihop 4
   neighbor OVERLAY send-community
   neighbor 10.1.0.0 remote-as 65001
   neighbor 10.2.0.0 remote-as 65001
   neighbor 172.16.0.2 peer group OVERLAY
   neighbor 172.16.0.2 remote-as 65021
   neighbor 172.16.0.3 peer group OVERLAY
   neighbor 172.16.0.3 remote-as 65031
   redistribute connected route-map LOOPBACKS
   !
   address-family evpn
      neighbor OVERLAY activate
   !
   address-family ipv4
      no neighbor OVERLAY activate
   !

```

```html

router bgp 65021
   maximum-paths 2 ecmp 2
   neighbor OVERLAY peer group
   neighbor OVERLAY update-source Loopback1
   neighbor OVERLAY ebgp-multihop 4
   neighbor OVERLAY send-community
   neighbor 10.1.0.4 remote-as 65001
   neighbor 10.2.0.3 remote-as 65001
   neighbor 172.16.0.1 peer group OVERLAY
   neighbor 172.16.0.1 remote-as 65011
   neighbor 172.16.0.3 peer group OVERLAY
   neighbor 172.16.0.3 remote-as 65031
   redistribute connected route-map LOOPBACKS
   !
   address-family evpn
      neighbor OVERLAY activate
   !
   address-family ipv4
      no neighbor OVERLAY activate

```

```html

router bgp 65031
   maximum-paths 2 ecmp 2
   neighbor OVERLAY peer group
   neighbor OVERLAY update-source Loopback1
   neighbor OVERLAY ebgp-multihop 4
   neighbor OVERLAY send-community
   neighbor 10.1.0.2 remote-as 65001
   neighbor 10.2.0.5 remote-as 65001
   neighbor 172.16.0.1 peer group OVERLAY
   neighbor 172.16.0.1 remote-as 65011
   neighbor 172.16.0.2 peer group OVERLAY
   neighbor 172.16.0.2 remote-as 65021
   redistribute connected route-map LOOPBACKS
   !
   address-family evpn
      neighbor OVERLAY activate
   !
   address-family ipv4
      no neighbor OVERLAY activate
   !

```

Проверяем с помощью команды:

```html

LEAF1#sh bgp summ
BGP summary information for VRF default
Router identifier 172.16.0.1, local AS number 65011
Neighbor            AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
---------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.1.0.0         65001 Established   IPv4 Unicast            Negotiated              3          3
10.2.0.0         65001 Established   IPv4 Unicast            Negotiated              3          3
172.16.0.2       65021 Established   L2VPN EVPN              Negotiated              4          4
172.16.0.3       65031 Established   L2VPN EVPN              Negotiated              4          4

```

Далее, настроим VLAN-based сервис для VXLAN L2, а также MAC-VRF, последовательно на  LEAF 1,2,3, .


```html

router bgp 65011
   !
   vlan 88
      rd 172.16.0.1:10088
      route-target both 1:10088
      redistribute learned
   !
   vlan 99
      rd 172.16.0.1:10099
      route-target both 1:10099
      redistribute learned
   !


```

```html

router bgp 65021
   vlan 77
      rd 172.16.0.2:10077
      route-target both 1:10077
      redistribute learned
   !
   vlan 99
      rd 172.16.0.2:10099
      route-target both 1:10099
      redistribute learned

```


```html

router bgp 65031
   vlan 77
      rd 172.16.0.3:10077
      route-target both 1:10077
      redistribute learned
   !
   vlan 88
      rd 172.16.0.3:10088
      route-target both 1:10088
      redistribute learned
   !
   vlan 99
      rd 172.16.0.3:10099
      route-target both 1:10099
      redistribute learned

```

Проверка на хосте:

```html

VPCS> sh ip

NAME        : VPCS[1]
IP/MASK     : 192.168.99.100/24
GATEWAY     : 192.168.99.1
DNS         : 
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.99.101

84 bytes from 192.168.99.101 icmp_seq=1 ttl=64 time=30.872 ms
84 bytes from 192.168.99.101 icmp_seq=2 ttl=64 time=14.241 ms
84 bytes from 192.168.99.101 icmp_seq=3 ttl=64 time=14.523 ms
^C
VPCS> 


```

**L3 VNI**

Лучше сразу настроить Assymetric IRB для будущего масштабирования и для выхода во внешние сети, а также виртуальный MAC-адрес.

LEAF1

```html


!
service routing protocols model multi-agent
!
hostname LEAF1

!
vrf instance IRB
!
cvx
   no shutdown
   !
   service vxlan
      no shutdown
      redistribute bgp evpn vxlan
!

!
interface Vlan77
   vrf IRB
   ip address virtual 192.168.77.1/24
!
interface Vlan88
   vrf IRB
   ip address virtual 192.168.88.1/24
!
interface Vlan99
   vrf IRB
   ip address virtual 192.168.99.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 77 vni 10077
   vxlan vlan 88 vni 10088
   vxlan vlan 99 vni 10099
   vxlan vrf IRB vni 10100
!
ip virtual-router mac-address 12:34:56:78:90:12
!
ip routing
ip routing vrf IRB

!
router bgp 65011
   !
   vlan 77
      rd 172.16.0.1:10077
      route-target both 1:10077
      redistribute learned
   !
   vlan 88
      rd 172.16.0.1:10077
      route-target both 1:10077
      redistribute learned
   !
   vlan 99
      rd 172.16.0.1:10099
      route-target both 1:10099
      redistribute learned
   !
   address-family evpn
      neighbor OVERLAY activate
   !
   address-family ipv4
      no neighbor OVERLAY activate
   !
   vrf IRB
      rd 172.16.0.1:10100
      route-target import evpn 1:10100
      route-target export evpn 1:10100
!
end

```

LEAF2

```html


service routing protocols model multi-agent
!
hostname LEAF2
!
spanning-tree mode mstp
!
vlan 77
!
vlan 99
   name VXLAN
!
vrf instance IRB
!
cvx
   no shutdown
   !
   service vxlan
      no shutdown
      redistribute bgp evpn vxlan
!

!
interface Vlan77
   vrf IRB
   ip address virtual 192.168.77.1/24
!
interface Vlan99
   vrf IRB
   ip address virtual 192.168.99.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 77 vni 10077
   vxlan vlan 99 vni 10099
   vxlan vrf IRB vni 10100
!
ip virtual-router mac-address 12:34:56:78:90:12
!
ip routing
ip routing vrf IRB

!
router bgp 65021

   !
   vlan 77
      rd 172.16.0.2:10077
      route-target both 1:10077
      redistribute learned
   !
   vlan 99
      rd 172.16.0.2:10099
      route-target both 1:10099
      redistribute learned
   !
   address-family evpn
      neighbor OVERLAY activate
   !
   address-family ipv4
      no neighbor OVERLAY activate
   !
   vrf IRB
      rd 172.16.0.2:10100
      route-target import evpn 1:10100
      route-target export evpn 1:10100
!
end

```


LEAF3

```html


!
service routing protocols model multi-agent
!
hostname LEAF3
!
spanning-tree mode mstp
!
vlan 77,88,99
!
vrf instance IRB
!
interface Vlan77
   vrf IRB
   ip address virtual 192.168.77.1/24
!
interface Vlan88
   vrf IRB
   ip address virtual 198.168.88.1/24
!
interface Vlan99
   vrf IRB
   ip address virtual 192.168.99.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 77 vni 10077
   vxlan vlan 88 vni 10088
   vxlan vrf IRB vni 10100
!
ip virtual-router mac-address 12:34:56:78:90:12
!
ip routing
ip routing vrf IRB
!

   !
   vrf IRB
      rd 172.16.0.3:10100
      route-target import evpn 1:10100
      route-target export evpn 1:10100
!
end

```

Проерить можно с тех же хостов, добавив VLAN 77,88:

```html

VPCS> sh ip

NAME        : VPCS[1]
IP/MASK     : 192.168.99.100/24
GATEWAY     : 192.168.99.1
DNS         : 
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.77.102

84 bytes from 192.168.77.102 icmp_seq=1 ttl=62 time=275.775 ms
84 bytes from 192.168.77.102 icmp_seq=2 ttl=62 time=16.975 ms
84 bytes from 192.168.77.102 icmp_seq=3 ttl=62 time=17.159 ms
84 bytes from 192.168.77.102 icmp_seq=4 ttl=62 time=17.138 ms
84 bytes from 192.168.77.102 icmp_seq=5 ttl=62 time=17.414 ms

VPCS> 

```

```html

VPCS> sh ip

NAME        : VPCS[1]
IP/MASK     : 192.168.77.102/24
GATEWAY     : 192.168.77.1
DNS         : 
MAC         : 00:50:79:66:68:0b
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.99.100

84 bytes from 192.168.99.100 icmp_seq=1 ttl=62 time=58.746 ms
84 bytes from 192.168.99.100 icmp_seq=2 ttl=62 time=21.099 ms
84 bytes from 192.168.99.100 icmp_seq=3 ttl=62 time=18.310 ms
^C
VPCS>

```

Диагностические данные:

LEAF1:

```html

LEAF1#sh bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.16.0.1, local AS number 65011
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 172.16.0.2:10099 mac-ip 0050.7966.6803
                                 172.16.0.2            -       100     0       65021 i
 *        RD: 172.16.0.2:10099 mac-ip 0050.7966.6803
                                 172.16.0.2            -       100     0       65031 65021 i
 * >      RD: 172.16.0.2:10099 mac-ip 0050.7966.6803 192.168.99.101
                                 172.16.0.2            -       100     0       65021 i
 *        RD: 172.16.0.2:10099 mac-ip 0050.7966.6803 192.168.99.101
                                 172.16.0.2            -       100     0       65031 65021 i
 * >      RD: 172.16.0.3:10088 mac-ip 0050.7966.6805
                                 172.16.0.3            -       100     0       65031 i
 *        RD: 172.16.0.3:10088 mac-ip 0050.7966.6805
                                 172.16.0.3            -       100     0       65021 65031 i
 * >      RD: 172.16.0.1:10099 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 172.16.0.1:10099 mac-ip 0050.7966.6808 192.168.99.100
                                 -                     -       -       0       i
 * >      RD: 172.16.0.1:10077 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >      RD: 172.16.0.1:10077 mac-ip 0050.7966.6809 192.168.77.100
                                 -                     -       -       0       i
 * >      RD: 172.16.0.3:10077 mac-ip 0050.7966.680a
                                 172.16.0.3            -       100     0       65031 i
 *        RD: 172.16.0.3:10077 mac-ip 0050.7966.680a
                                 172.16.0.3            -       100     0       65021 65031 i
 * >      RD: 172.16.0.3:10077 mac-ip 0050.7966.680a 192.168.77.101
                                 172.16.0.3            -       100     0       65031 i
 *        RD: 172.16.0.3:10077 mac-ip 0050.7966.680a 192.168.77.101
                                 172.16.0.3            -       100     0       65021 65031 i
 * >      RD: 172.16.0.2:10077 mac-ip 0050.7966.680b
                                 172.16.0.2            -       100     0       65021 i
 *        RD: 172.16.0.2:10077 mac-ip 0050.7966.680b
                                 172.16.0.2            -       100     0       65031 65021 i
 * >      RD: 172.16.0.2:10077 mac-ip 0050.7966.680b 192.168.77.102
                                 172.16.0.2            -       100     0       65021 i
 *        RD: 172.16.0.2:10077 mac-ip 0050.7966.680b 192.168.77.102
                                 172.16.0.2            -       100     0       65031 65021 i
LEAF1#
LEAF1#

```


LEAF2:

```html

LEAF2#sh bgp evpn route-type mac-ip 192.168.77.101 de
BGP routing table information for VRF default
Router identifier 172.16.0.2, local AS number 65021
BGP routing table entry for mac-ip 0050.7966.680a 192.168.77.101, Route Distinguisher: 172.16.0.3:10077
 Paths: 2 available
  65031
    172.16.0.3 from 172.16.0.3 (172.16.0.3)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:1:10077 Route-Target-AS:1:10100 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:f6:ad:37
      VNI: 10077 L3 VNI: 10100 ESI: 0000:0000:0000:0000:0000
  65011 65031
    172.16.0.3 from 172.16.0.1 (172.16.0.1)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external
      Extended Community: Route-Target-AS:1:10077 Route-Target-AS:1:10100 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:f6:ad:37
      VNI: 10077 L3 VNI: 10100 ESI: 0000:0000:0000:0000:0000
LEAF2#

```

