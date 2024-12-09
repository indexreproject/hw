Домашнее задание №4

Underlay. BGP

Цель:

 - Настроить BGP для Underlay сети

Описание:

 - Настроите BGP в Underlay сети, для IP связанности между всеми сетевыми устройствами. iBGP или eBGP.
 - Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройст
 - Убедитесь в наличии IP связанности между устройствами в BGP домене

Схема сети:

![CLOS](CLOS_IP+Lo.png)

**iBGP.**

SPINE 1 и 2 будут выступать в роли RR, у которых настроены cluster-id 1 и 2, соответственно.

Последовательная конфигурация коммутаторов:

SPINE1

```html

interface LoopBack0
 ip address 10.100.1.1 255.255.255.0
#
interface LoopBack1
 ip address 172.31.0.1 255.255.255.255
#
bgp 65001
 router-id 172.31.0.1
 peer 10.1.0.1 as-number 65001
 peer 10.1.0.3 as-number 65001
 peer 10.1.0.5 as-number 65001
 #
 ipv4-family unicast
  reflector cluster-id 1
  network 10.1.0.0 255.255.255.254
  network 10.1.0.2 255.255.255.254
  network 10.1.0.4 255.255.255.254
  peer 10.1.0.1 enable
  peer 10.1.0.1 reflect-client
  peer 10.1.0.3 enable
  peer 10.1.0.3 reflect-client
  peer 10.1.0.5 enable
  peer 10.1.0.5 reflect-client
#


```

SPINE2

```html

interface LoopBack0
 ip address 10.100.1.2 255.255.255.0
#
interface LoopBack1
 ip address 172.31.0.2 255.255.255.255
#
bgp 65001
 router-id 172.31.0.2
 peer 10.2.0.1 as-number 65001
 peer 10.2.0.2 as-number 65001
 peer 10.2.0.4 as-number 65001
 #
 ipv4-family unicast
  reflector cluster-id 2
  network 10.2.0.0 255.255.255.254
  network 10.2.0.2 255.255.255.254
  network 10.2.0.4 255.255.255.254
  peer 10.2.0.1 enable
  peer 10.2.0.1 reflect-client
  peer 10.2.0.2 enable
  peer 10.2.0.2 reflect-client
  peer 10.2.0.4 enable
  peer 10.2.0.4 reflect-client
#


```
