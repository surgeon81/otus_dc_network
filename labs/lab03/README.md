# Проектирование адресного пространства

### Цели
- Собрать топологию CLOS
- Распределить адресное пространство для Underlay сети

### Топология сети
![CLOS_network.png](CLOS_network.png)

### Таблицы адресного пространства

#### Общая таблица

|Уровень|Подсеть|Маска|Назначение|Заметки
|---|---|---|---|---|
0|10.0.0.0|16|SSuper Spine|Зарезервировано
1|10.1.0.0|16|Super Spine|Зарезервировано
2|10.2.0.0|16|Spine
3|10.3.0.0|16|Leaf

#### Распределение подсетей по SSuper Spine - Level 0 (резерв)
|SSS0|SSS1|SSS2|...|SSS63|Маска|Назначение
|---|---|---|---|---|---|---|
10.0.0.0|10.0.4.0|10.0.8.0|...|10.0.252.0|24|Lo underlay
10.0.1.0|10.0.5.0|10.0.9.0|...|10.0.253.0|24|Lo overlay
10.0.2.0|10.0.6.0|10.0.10.0|...|10.0.254.0|24|p2p1
10.0.3.0|10.0.7.0|10.0.11.0|...|10.0.255.0|24|p2p2


#### Распределение подсетей для Super Spine - Level 1 (резерв)
|SS0|SS1|SS2|...|SS63|Маска|Назначение
|---|---|---|---|---|---|---|
10.1.0.0|10.1.4.0|10.1.8.0|...|10.1.252.0|24|Lo underlay
10.1.1.0|10.1.5.0|10.1.9.0|...|10.1.253.0|24|Lo overlay
10.1.2.0|10.1.6.0|10.1.10.0|...|10.1.254.0|24|p2p1
10.1.3.0|10.1.7.0|10.1.11.0|...|10.1.255.0|24|p2p2

#### Распределение подсетей для Spine (Level 2)
|S0|S1|S2|...|S63|Маска|Назначение
|---|---|---|---|---|---|---|
10.2.0.0|10.2.4.0|10.2.8.0|...|10.2.252.0|24|Lo underlay
10.2.1.0|10.2.5.0|10.2.9.0|...|10.2.253.0|24|Lo overlay
10.2.2.0|10.2.6.0|10.2.10.0|...|10.2.254.0|24|p2p1
10.2.3.0|10.2.7.0|10.2.11.0|...|10.2.255.0|24|p2p2

#### Распределение подсетей для Leaf (Level 3)
|Leaf0|Leaf1|Leaf2|...|Leaf63|Маска|Назначение
|---|---|---|---|---|---|---|
10.3.0.0|10.3.4.0|10.3.8.0|...|10.3.252.0|24|Lo underlay
10.3.1.0|10.3.5.0|10.3.9.0|...|10.3.253.0|24|Lo overlay
10.3.2.0|10.3.6.0|10.3.10.0|...|10.3.254.0|24|App1
10.3.3.0|10.3.7.0|10.3.11.0|...|10.3.255.0|24|App2

#### Адреса коммутатора Spine 0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.0.0|32|Loopback 0|Lo underlay
10.2.1.0|32|Loopback 100|Lo overlay
10.2.2.1|31|Eth 1|p2p leaf 0
10.2.2.3|31|Eth 2|p2p leaf 1
10.2.2.5|31|Eth 3|p2p leaf 2

#### Адреса коммутатора Spine 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.4.0|32|Loopback 0|Lo underlay
10.2.5.0|32|Loopback 100|Lo overlay
10.2.6.1|31|Eth 1|p2p leaf 0
10.2.6.3|31|Eth 2|p2p leaf 1
10.2.6.5|31|Eth 3|p2p leaf 2


#### Адреса коммутатора Leaf 0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.0.0|32|Loopback 0|Lo underlay
10.3.1.0|32|Loopback 100|Lo overlay
10.2.2.0|31|Eth 9|p2p spine 0
10.2.6.0|31|Eth 10|p2p spine 1
10.3.2.254|24|VLAN 10|App1

#### Адреса коммутатора Leaf 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.4.0|32|Loopback 0|Lo underlay
10.3.5.0|32|Loopback 100|Lo overlay
10.2.2.2|31|Eth 9|p2p spine 0
10.2.6.2|31|Eth 10|p2p spine 1
10.3.6.254|24|VLAN 10|App1

#### Адреса коммутатора Leaf 2
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.8.0|32|Loopback 0|Lo underlay
10.3.9.0|32|Loopback 100|Lo overlay
10.2.2.4|31|Eth 9|p2p spine 0
10.2.6.4|31|Eth 10|p2p spine 1
10.3.10.254|24|VLAN 10|App1

#### Данные подключения клиентов
Клиент|Leaf sw|Port|IP|Gateway|Mask
|---|---|---|---|---|---|
Client1|Leaf_0|1|10.3.2.1|10.3.2.254|24
Client2|Leaf_1|1|10.3.6.1|10.3.6.254|24
Client3|Leaf_2|1|10.3.10.1|10.3.10.254|24
Clietn4|Leaf_2|2|10.3.10.2|10.3.10.254|24


### Подготовка стенда

#### [Настройка Spine_0](Spine_0.cfg)

```
interface Ethernet1
   description Leaf_0:Eth9
   no switchport
   ip address 10.2.2.1/31
!
interface Ethernet2
   description Leaf_1:Eth9
   no switchport
   ip address 10.2.2.3/31
!
interface Ethernet3
   description Leaf_2:Eth9
   no switchport
   ip address 10.2.2.5/31
!
interface Loopback0
   ip address 10.2.0.0/32
!
interface Loopback100
   ip address 10.2.1.0/32
```   
#### [Настройка Spine_1](Spine_1.cfg)

```
interface Ethernet1
   description Leaf_0:Eth10
   no switchport
   ip address 10.2.6.1/31
!
interface Ethernet2
   description Leaf_1:Eth10
   no switchport
   ip address 10.2.6.3/31
!
interface Ethernet3
   description Leaf_2:Eth10
   no switchport
   ip address 10.2.6.5/31
!
interface Loopback0
   ip address 10.2.4.0/32
!
interface Loopback100
   ip address 10.2.5.0/32
!
```

#### [Настройка Leaf_0](Leaf_0.cfg)

```
interface Ethernet1
   description Client_1:Eth0
   switchport access vlan 10
!
interface Ethernet9
   description Spine_0:Eth1
   no switchport
   ip address 10.2.2.0/31
!
interface Ethernet10
   description Spine_1:Eth1
   no switchport
   ip address 10.2.6.0/31
!
interface Loopback0
   ip address 10.3.0.0/32
!
interface Loopback100
   ip address 10.3.1.0/32
!
interface Vlan10
   description App1
   ip address 10.3.2.254/24
   ```

 #### [Настройка Leaf_1](Leaf_1.cfg)

 ```
 interface Ethernet1
   description Client_2:Eth0
   switchport access vlan 10
!
interface Ethernet9
   description Spine_0:Eth2
   no switchport
   ip address 10.2.2.2/31
!
interface Ethernet10
   description Spine_1:Eth2
   no switchport
   ip address 10.2.6.2/31
!
interface Loopback0
   ip address 10.3.4.0/32
!
interface Loopback100
   ip address 10.3.5.0/32
!
interface Vlan10
   description App1
   ip address 10.3.6.254/24
```

 #### [Настройка Leaf_2](Leaf_2.cfg)

 ```
 nterface Ethernet1
   description Client_3:Eth0
   switchport access vlan 10
!
interface Ethernet2
   description Client_4:Eth0
   switchport access vlan 10
!
interface Ethernet9
   description Spine_0:Eth3
   no switchport
   ip address 10.2.2.4/31
!
interface Ethernet10
   description Spine_1:Eth3
   no switchport
   ip address 10.2.6.4/31
!
interface Loopback0
   ip address 10.3.8.0/32
!
interface Loopback100
   ip address 10.3.9.0/32
!
interface Vlan10
   description App1
   ip address 10.3.10.254/24
```
