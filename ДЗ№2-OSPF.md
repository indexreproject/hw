Домашнее задание №1

Underlay. OSPF

Цель:

Настроить OSPF для Underlay сети

Описание:

Настроить OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами.
Зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
Убедится в наличии IP связанности между устройствами в OSFP домене.


Схема сети:

![CLOS](CLOS.png)


Последовательная конфигурация коммутаторов:

**LEAF1**

```html
interface GE1/0/1
 undo portswitch
 undo shutdown
 ip address 10.1.0.1 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface GE1/0/2
 undo portswitch
 undo shutdown
 ip address 10.2.0.1 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface LoopBack1
 ip address 172.16.0.2 255.255.255.255
 ospf enable 100 area 0.0.0.0
#
interface NULL0
#
ospf 100 router-id 172.16.0.2
 area 0.0.0.0
#
```

**LEAF2**

```html
interface GE1/0/3
 undo portswitch
 undo shutdown
 ip address 10.1.0.3 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface GE1/0/4
 undo portswitch
 undo shutdown
 ip address 10.2.0.2 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface LoopBack1
 ip address 172.16.0.1 255.255.255.255
 ospf enable 100 area 0.0.0.0
#
interface NULL0
#
ospf 100 router-id 172.16.0.1
 area 0.0.0.0
#
```

**LEAF3**

```html
interface GE1/0/5
 undo portswitch
 undo shutdown
 ip address 10.1.0.5 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface GE1/0/6
 undo portswitch
 undo shutdown
 ip address 10.2.0.4 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface LoopBack1
 ip address 172.16.0.3 255.255.255.255
 ospf enable 100 area 0.0.0.0
#
interface NULL0
#
ospf 100 router-id 172.16.0.3
 area 0.0.0.0
#
```

**SPINE1**

```html
interface GE1/0/1
 undo portswitch
 undo shutdown
 ip address 10.1.0.0 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface GE1/0/2
 shutdown
#
interface GE1/0/3
 undo portswitch
 undo shutdown
 ip address 10.1.0.2 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface GE1/0/4
 shutdown
#
interface GE1/0/5
 undo portswitch
 undo shutdown
 ip address 10.1.0.4 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface LoopBack1
 ip address 172.31.0.1 255.255.255.255
 ospf enable 100 area 0.0.0.0
#
interface NULL0
#
ospf 100 router-id 172.31.0.1
 area 0.0.0.0
#
```
**SPINE2**

```html
interface GE1/0/2
 undo portswitch
 undo shutdown
 ip address 10.2.0.0 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface GE1/0/3
 shutdown
#
interface GE1/0/4
 undo portswitch
 undo shutdown
 ip address 10.2.0.3 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface GE1/0/5
 shutdown
#
interface GE1/0/6
 undo portswitch
 undo shutdown
 ip address 10.2.0.5 255.255.255.254
 ospf network-type p2p
 ospf enable 100 area 0.0.0.0
#
interface LoopBack1
 ip address 172.31.0.2 255.255.255.255
 ospf enable 100 area 0.0.0.0
#
interface NULL0
#
ospf 100 router-id 172.31.0.2
 area 0.0.0.0
#
```

*Соседство установлено*

**SPINE1**

```html

<SPINE1>disp ospf peer br
OSPF Process 100 with Router ID 172.31.0.1
                   Peer Statistic Information
Total number of peer(s): 3       
 Peer(s) in full state: 3       
-----------------------------------------------------------------------------
 Area Id         Interface                  Neighbor id          State       
 0.0.0.0         GE1/0/1                    172.16.0.2           Full        
 0.0.0.0         GE1/0/3                    172.16.0.1           Full        
 0.0.0.0         GE1/0/5                    172.16.0.3           Full        
-----------------------------------------------------------------------------

```

**SPINE2**

```html

<SPINE2>disp ospf peer br
OSPF Process 100 with Router ID 172.31.0.2
                   Peer Statistic Information
Total number of peer(s): 2       
 Peer(s) in full state: 2       
-----------------------------------------------------------------------------
 Area Id         Interface                  Neighbor id          State       
 0.0.0.0         GE1/0/2                    172.16.0.2           Full        
 0.0.0.0         GE1/0/6                    172.16.0.3           Full        
-----------------------------------------------------------------------------

```

