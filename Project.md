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


