# Построение Underlay сети(OSPF)

### Цели
- Настроить OSPF для Underlay сети


### Топология сети
![CLOS_OSPF.png](/labs/lab05/CLOS_OSPF.png)

### Описание топологии
1. Все интерфейсы комутаторов всех уровней Spine находятся в Area 0
2. Все коммутаторы уровня Leaf являются ABR
   1. Интерфейсы, подключаемые к уровню Spine, принадлежат Area 0
   2. Все остальные интерфейсы принадлежат своей Area
3. Номером Area для коммутаторов уровня Leaf является IP адрес его интерфейса Loopback0
4. OSPF Router ID коммутаторов всех уровней является IP адрес его интерфейса Loopback0
5. Все non-backbon area - являются NSSA
6. Loopback0 интерфейсы не анонсируются на всех уровнях
7. Интерфейсы >= Loopback100 анонсируются только на уровне Leaf
8. Анонс >=Loopback100 интефейсов на уровнях Spine включается только в случаях терминации VxLAN туннеля на данном уровне
9. На всех p2p линках с подсетью /31 используется функция "netwok type point-to-point"
10. На всех Leaf необходимо включать отправку суммированного маршрута командой "area x.x.x.x range y.y.y.y/22", где y.y.y.y/22 - префикс, выделенный для данного Leaf

### Адреса устройств

#### Адреса коммутатора Spine 0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.0.0|32|Loopback 0|RID
10.2.1.0|32|Loopback 100|Reserved
10.2.2.1|31|Eth 1|p2p leaf 0
10.2.2.3|31|Eth 2|p2p leaf 1
10.2.2.5|31|Eth 3|p2p leaf 2

#### Адреса коммутатора Spine 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.4.0|32|Loopback 0|RID
10.2.5.0|32|Loopback 100|Reserved
10.2.6.1|31|Eth 1|p2p leaf 0
10.2.6.3|31|Eth 2|p2p leaf 1
10.2.6.5|31|Eth 3|p2p leaf 2


#### Адреса коммутатора Leaf 0
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.0.0|32|Loopback 0|10.3.0.0|RID, Area
10.3.1.0|32|Loopback 100|10.3.0.0|Lo overlay
10.3.2.254|24|VLAN 10|10.3.0.0|App1
10.2.2.0|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.0|31|Eth 10|0.0.0.0|p2p spine 1

#### Адреса коммутатора Leaf 1
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.4.0|32|Loopback 0|10.3.4.0|RID, Area
10.3.5.0|32|Loopback 100|10.3.4.0|Lo overlay
10.3.6.254|24|VLAN 10|10.3.4.0|App1
10.2.2.2|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.2|31|Eth 10|0.0.0.0|p2p spine 1

#### Адреса коммутатора Leaf 2
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.8.0|32|Loopback 0|10.3.8.0|RID, Area
10.3.9.0|32|Loopback 100|10.3.8.0|Lo overlay
10.3.10.254|24|VLAN 10|10.3.8.0|App1
10.2.2.4|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.4|31|Eth 10|0.0.0.0|p2p spine 1

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
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 7O4CGaiAcR0=
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Leaf_1:Eth9
   no switchport
   ip address 10.2.2.3/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Leaf_2:Eth9
   no switchport
   ip address 10.2.2.5/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.2.0.0/32
!
interface Loopback100
   ip address 10.2.1.0/32
!
ip routing
!
router ospf 1
   router-id 10.2.0.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!

```   
#### [Настройка Spine_1](Spine_1.cfg)

```
interface Ethernet1
   description Leaf_0:Eth10
   no switchport
   ip address 10.2.6.1/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 7O4CGaiAcR0=
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description Leaf_1:Eth10
   no switchport
   ip address 10.2.6.3/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description Leaf_2:Eth10
   no switchport
   ip address 10.2.6.5/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.2.4.0/32
!
interface Loopback100
   ip address 10.2.5.0/32
!
ip routing
!
router ospf 1
   router-id 10.2.4.0
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
```

#### [Настройка Leaf_0](Leaf_0.cfg)

```
vlan 10
   name App1
!
interface Ethernet1
   description Client_1:Eth0
   switchport access vlan 10
!
interface Ethernet9
   description Spine_0:Eth1
   no switchport
   ip address 10.2.2.0/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 RdTlHg2Ijiw=
   ip ospf area 0.0.0.0
!
interface Ethernet10
   description Spine_1:Eth1
   no switchport
   ip address 10.2.6.0/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 TSPxiC76NC0=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.3.0.0/32
!
interface Loopback100
   ip address 10.3.1.0/32
   ip ospf area 10.3.0.0
!
interface Vlan10
   description App1
   ip address 10.3.2.254/24
   ip ospf area 10.3.0.0
!
ip routing
!
router ospf 1
   router-id 10.3.0.0
   passive-interface default
   no passive-interface Ethernet9
   no passive-interface Ethernet10
   area 10.3.0.0 nssa no-summary
   area 10.3.0.0 range 10.3.0.0/22
   max-lsa 12000
   ```

 #### [Настройка Leaf_1](Leaf_1.cfg)

 ```
 vlan 10
   name App1
!
interface Ethernet1
   description Client_2:Eth0
   switchport access vlan 10
!
interface Ethernet9
   description Spine_0:Eth2
   no switchport
   ip address 10.2.2.2/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 RdTlHg2Ijiw=
   ip ospf area 0.0.0.0
!
interface Ethernet10
   description Spine_1:Eth2
   no switchport
   ip address 10.2.6.2/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 TSPxiC76NC0=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.3.4.0/32
!
interface Loopback100
   ip address 10.3.5.0/32
   ip ospf area 10.3.4.0
!
interface Vlan10
   description App1
   ip address 10.3.6.254/24
   ip ospf area 10.3.4.0
!
ip routing
!
router ospf 1
   router-id 10.3.4.0
   passive-interface default
   no passive-interface Ethernet9
   no passive-interface Ethernet10
   area 10.3.4.0 nssa no-summary
   area 10.3.4.0 range 10.3.4.0/22
   max-lsa 12000
!
```

 #### [Настройка Leaf_2](Leaf_2.cfg)

 ```
 vlan 10
   name App1
!
interface Ethernet1
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
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 RdTlHg2Ijiw=
   ip ospf area 0.0.0.0
!
interface Ethernet10
   description Spine_1:Eth3
   no switchport
   ip address 10.2.6.4/31
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 TSPxiC76NC0=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.3.8.0/32
!
interface Loopback100
   ip address 10.3.9.0/32
   ip ospf area 10.3.8.0
!
interface Vlan10
   description App1
   ip address 10.3.10.254/24
   ip ospf area 10.3.8.0
!
ip routing
!
router ospf 1
   router-id 10.3.8.0
   passive-interface default
   no passive-interface Ethernet9
   no passive-interface Ethernet10
   area 10.3.8.0 nssa no-summary
   area 10.3.8.0 range 10.3.8.0/22
   max-lsa 12000
```
