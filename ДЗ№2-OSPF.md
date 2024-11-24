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
 0.0.0.0         GE1/0/4                    172.16.0.1           Full        
 0.0.0.0         GE1/0/6                    172.16.0.3           Full        
-----------------------------------------------------------------------------

```

*Проверки связаности*

SPINE1>SPINE1

```html
interface LoopBack1
 ip address 172.31.0.1 255.255.255.255
 ospf enable 100 area 0.0.0.0
#
return
<SPINE1>ping -a 172.31.0.1 172.31.0.2
  PING 172.31.0.2: 56  data bytes, press CTRL_C to break
    Reply from 172.31.0.2: bytes=56 Sequence=1 ttl=254 time=12 ms
    Reply from 172.31.0.2: bytes=56 Sequence=2 ttl=254 time=5 ms
    Reply from 172.31.0.2: bytes=56 Sequence=3 ttl=254 time=10 ms
    Reply from 172.31.0.2: bytes=56 Sequence=4 ttl=254 time=14 ms
    Reply from 172.31.0.2: bytes=56 Sequence=5 ttl=254 time=9 ms

  --- 172.31.0.2 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 5/10/14 ms

```

**SPINE1>LEAF1**
**SPINE1>LEAF2**
**SPINE1>LEAF1**

```html
<SPINE1>ping -a 172.31.0.1 172.16.0.1
  PING 172.16.0.1: 56  data bytes, press CTRL_C to break
    Reply from 172.16.0.1: bytes=56 Sequence=1 ttl=255 time=11 ms
    Reply from 172.16.0.1: bytes=56 Sequence=2 ttl=255 time=5 ms
    Reply from 172.16.0.1: bytes=56 Sequence=3 ttl=255 time=4 ms
    Reply from 172.16.0.1: bytes=56 Sequence=4 ttl=255 time=7 ms
    Reply from 172.16.0.1: bytes=56 Sequence=5 ttl=255 time=4 ms

  --- 172.16.0.1 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 4/6/11 ms
 
<SPINE1>ping -a 172.31.0.1 172.16.0.2
  PING 172.16.0.2: 56  data bytes, press CTRL_C to break
    Reply from 172.16.0.2: bytes=56 Sequence=1 ttl=255 time=7 ms
    Reply from 172.16.0.2: bytes=56 Sequence=2 ttl=255 time=4 ms
    Reply from 172.16.0.2: bytes=56 Sequence=3 ttl=255 time=3 ms
    Reply from 172.16.0.2: bytes=56 Sequence=4 ttl=255 time=3 ms
    Reply from 172.16.0.2: bytes=56 Sequence=5 ttl=255 time=6 ms

  --- 172.16.0.2 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 3/4/7 ms
 
<SPINE1>ping -a 172.31.0.1 172.16.0.3
  PING 172.16.0.3: 56  data bytes, press CTRL_C to break
    Reply from 172.16.0.3: bytes=56 Sequence=1 ttl=255 time=8 ms
    Reply from 172.16.0.3: bytes=56 Sequence=2 ttl=255 time=3 ms
    Reply from 172.16.0.3: bytes=56 Sequence=3 ttl=255 time=6 ms
    Reply from 172.16.0.3: bytes=56 Sequence=4 ttl=255 time=4 ms
    Reply from 172.16.0.3: bytes=56 Sequence=5 ttl=255 time=5 ms

  --- 172.16.0.3 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 3/5/8 ms

```



**SPINE2>LEAF1**
**SPINE2>LEAF2**
**SPINE2>LEAF3**

```html

<SPINE2>dis cur int lo1
#
interface LoopBack1
 ip address 172.31.0.2 255.255.255.255
 ospf enable 100 area 0.0.0.0
#
return
<SPINE2>ping -a 172.31.0.2 172.16.0.1
  PING 172.16.0.1: 56  data bytes, press CTRL_C to break
    Reply from 172.16.0.1: bytes=56 Sequence=1 ttl=255 time=10 ms
    Reply from 172.16.0.1: bytes=56 Sequence=2 ttl=255 time=4 ms
    Reply from 172.16.0.1: bytes=56 Sequence=3 ttl=255 time=3 ms
    Reply from 172.16.0.1: bytes=56 Sequence=4 ttl=255 time=7 ms
    Reply from 172.16.0.1: bytes=56 Sequence=5 ttl=255 time=5 ms

  --- 172.16.0.1 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 3/5/10 ms
 
<SPINE2>ping -a 172.31.0.2 172.16.0.2
  PING 172.16.0.2: 56  data bytes, press CTRL_C to break
    Reply from 172.16.0.2: bytes=56 Sequence=1 ttl=255 time=5 ms
    Reply from 172.16.0.2: bytes=56 Sequence=2 ttl=255 time=6 ms
    Reply from 172.16.0.2: bytes=56 Sequence=3 ttl=255 time=2 ms
    Reply from 172.16.0.2: bytes=56 Sequence=4 ttl=255 time=5 ms
    Reply from 172.16.0.2: bytes=56 Sequence=5 ttl=255 time=14 ms

  --- 172.16.0.2 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 2/6/14 ms
 
<SPINE2>ping -a 172.31.0.2 172.16.0.3
  PING 172.16.0.3: 56  data bytes, press CTRL_C to break
    Reply from 172.16.0.3: bytes=56 Sequence=1 ttl=255 time=6 ms
    Reply from 172.16.0.3: bytes=56 Sequence=2 ttl=255 time=3 ms
    Reply from 172.16.0.3: bytes=56 Sequence=3 ttl=255 time=4 ms
    Reply from 172.16.0.3: bytes=56 Sequence=4 ttl=255 time=5 ms
    Reply from 172.16.0.3: bytes=56 Sequence=5 ttl=255 time=3 ms

  --- 172.16.0.3 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 3/4/6 ms
    
```

*Вывод топологии*

**SPINE1**

```html

<SPINE1>dis ospf top

          OSPF Process 100 with Router ID 172.31.0.1
Bits :
B - ABR    E - ASBR    V - VIRTUAL    NT - NSSA Translator

OSPF Area 0.0.0.0 topology
----------------------------------------------------------------
Type  ID                  Bits Metric  Next-Hop        Interface
Rtr   172.16.0.1               1       10.1.0.3        GE1/0/3        
Rtr   172.16.0.2               1       10.1.0.1        GE1/0/1        
Rtr   172.16.0.3               1       10.1.0.5        GE1/0/5        
Rtr   172.31.0.1               --
Rtr   172.31.0.2               2       10.1.0.5        GE1/0/5        
                                       10.1.0.3        GE1/0/3        
                                       10.1.0.1        GE1/0/1

```


**SPINE2**

```html

<SPINE2>dis ospf to

          OSPF Process 100 with Router ID 172.31.0.2
Bits :
B - ABR    E - ASBR    V - VIRTUAL    NT - NSSA Translator

OSPF Area 0.0.0.0 topology
----------------------------------------------------------------
Type  ID                  Bits Metric  Next-Hop        Interface
Rtr   172.16.0.1               1       10.2.0.2        GE1/0/4        
Rtr   172.16.0.2               1       10.2.0.1        GE1/0/2        
Rtr   172.16.0.3               1       10.2.0.4        GE1/0/6        
Rtr   172.31.0.1               2       10.2.0.4        GE1/0/6        
                                       10.2.0.2        GE1/0/4        
                                       10.2.0.1        GE1/0/2        
Rtr   172.31.0.2               --

```
