Домашнее задание №3

Underlay. ISIS

Цель:

 - Настроить ISIS для Underlay сети

Описание:

 - Настроить ISIS в Underlay сети, для IP связанности между всеми сетевыми устройствами.
 - Зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
 - Убедится в наличии IP связанности между устройствами в ISIS домене.


Схема сети:

![CLOS](CLOS.png)


Последовательная конфигурация коммутаторов (всё связанное с OSPF удалено):


**LEAF1**

```html
isis 1
 is-level level-2
 network-entity 21.0000.0000.0011.00
#
interface MEth0/0/0
 undo shutdown
#
interface GE1/0/0
 shutdown
#
interface GE1/0/1
 undo portswitch
 undo shutdown
 ip address 10.1.0.0 255.255.255.254
 isis enable 1
 isis circuit-type p2p
#
interface GE1/0/2
 shutdown
#
interface GE1/0/3
 undo portswitch
 undo shutdown
 ip address 10.1.0.2 255.255.255.254
 isis enable 1
 isis circuit-type p2p
#
interface GE1/0/4
 shutdown
#
interface GE1/0/5
 undo portswitch
 undo shutdown
 ip address 10.1.0.4 255.255.255.254
 isis enable 1
 isis circuit-type p2p
#

```

**LEAF2**

```html
isis 1
 is-level level-2
 network-entity 21.0000.0000.0012.00
#
interface MEth0/0/0
 undo shutdown
#
interface GE1/0/0
 shutdown
#
interface GE1/0/1
 shutdown
#
interface GE1/0/2
 undo portswitch
 undo shutdown
 ip address 10.2.0.0 255.255.255.254
 isis enable 1
 isis circuit-type p2p
#
interface GE1/0/3
 shutdown
#
interface GE1/0/4
 undo portswitch
 undo shutdown
 ip address 10.2.0.3 255.255.255.254
 isis enable 1
 isis circuit-type p2p
#
interface GE1/0/5
 shutdown
#
interface GE1/0/6
 undo portswitch
 undo shutdown
 ip address 10.2.0.5 255.255.255.254
 isis enable 1
 isis circuit-type p2p
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
