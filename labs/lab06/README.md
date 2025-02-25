# Построение Underlay сети(ISIS)

### Цели
- Настроить ISIS для Underlay сети


### Топология сети
![CLOS_ISIS.png](/labs/lab06/CLOS_ISIS.png)

### Описание топологии
1. Все интерфейсы комутаторов всех уровней Spine находятся в Area 9999
2. Вск коммутаторы уровней Spine относятся к ISIS L2 коммутаторам
2. Все коммутаторы уровня Leaf являются L1/L2
3. Все коммутаторы уровня Leaf пренадлежат своей Area, номер которой равен последнему октету адреса интерфейса Looback0
4. ISIS Router NET коммутаторов всех уровней является видоизмененный IP адрес его интерфейса Loopback0
6. Loopback0 интерфейсы не анонсируются на всех уровнях
7. Интерфейсы >= Loopback100 анонсируются только на уровне Leaf
8. Анонс >=Loopback100 интефейсов на уровнях Spine включается только в случаях терминации VxLAN туннеля на данном уровне
9. На всех p2p линках с подсетью /31 используется функция "netwok type point-to-point"
10. На всех коммутаторах необходимо включать "advertise passive-only" для анонса Looback и SVI адресов

### Адреса устройств

#### Адреса коммутатора Spine 0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.0.0|32|Loopback 0|NET
10.2.1.0|32|Loopback 100|Reserved
10.2.2.1|31|Eth 1|p2p leaf 0
10.2.2.3|31|Eth 2|p2p leaf 1
10.2.2.5|31|Eth 3|p2p leaf 2

#### Адреса коммутатора Spine 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.4.0|32|Loopback 0|NET
10.2.5.0|32|Loopback 100|Reserved
10.2.6.1|31|Eth 1|p2p leaf 0
10.2.6.3|31|Eth 2|p2p leaf 1
10.2.6.5|31|Eth 3|p2p leaf 2


#### Адреса коммутатора Leaf 0
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.0.0|32|Loopback 0|10.3.0.0|NET
10.3.1.0|32|Loopback 100|10.3.0.0|Lo overlay
10.3.2.254|24|VLAN 10|10.3.0.0|App1
10.2.2.0|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.0|31|Eth 10|0.0.0.0|p2p spine 1

#### Адреса коммутатора Leaf 1
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.4.0|32|Loopback 0|10.3.4.0|NET
10.3.5.0|32|Loopback 100|10.3.4.0|Lo overlay
10.3.6.254|24|VLAN 10|10.3.4.0|App1
10.2.2.2|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.2|31|Eth 10|0.0.0.0|p2p spine 1

#### Адреса коммутатора Leaf 2
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.8.0|32|Loopback 0|10.3.8.0|NET
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
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Ethernet2
   description Leaf_1:Eth9
   no switchport
   ip address 10.2.2.3/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Ethernet3
   description Leaf_2:Eth9
   no switchport
   ip address 10.2.2.5/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Loopback0
   ip address 10.2.0.0/32
!
interface Loopback100
   ip address 10.2.1.0/32
!
ip routing
!
router isis 1
   net 49.9999.0100.0200.0000.00
   is-type level-2
   advertise passive-only
   authentication mode md5
   authentication key 7 btRsZaMBerY=
   !
   address-family ipv4 unicast
!

```   
#### [Настройка Spine_1](Spine_1.cfg)

```
interface Ethernet1
   description Leaf_0:Eth10
   no switchport
   ip address 10.2.6.1/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Ethernet2
   description Leaf_1:Eth10
   no switchport
   ip address 10.2.6.3/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Ethernet3
   description Leaf_2:Eth10
   no switchport
   ip address 10.2.6.5/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Loopback0
   ip address 10.2.4.0/32
!
interface Loopback100
   ip address 10.2.5.0/32
!
ip routing
!
router isis 1
   net 49.9999.0100.0200.4000.00
   is-type level-2
   advertise passive-only
   authentication mode md5
   authentication key 7 btRsZaMBerY=
   !
   address-family ipv4 unicast
!
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
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Ethernet10
   description Spine_1:Eth1
   no switchport
   ip address 10.2.6.0/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Loopback0
   ip address 10.3.0.0/32
!
interface Loopback100
   ip address 10.3.1.0/32
   isis enable 1
   isis passive
!
interface Management1
!
interface Vlan10
   description App1
   ip address 10.3.2.254/24
   isis enable 1
   isis passive
!
ip routing
!
router isis 1
   net 49.0000.0100.0300.0000.00
   advertise passive-only
   authentication mode md5
   authentication key 7 btRsZaMBerY=
   !
   address-family ipv4 unicast
!
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
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Ethernet10
   description Spine_1:Eth2
   no switchport
   ip address 10.2.6.2/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Loopback0
   ip address 10.3.4.0/32
!
interface Loopback100
   ip address 10.3.5.0/32
   isis enable 1
   isis passive
!
interface Management1
!
interface Vlan10
   description App1
   ip address 10.3.6.254/24
   isis enable 1
   isis passive
!
ip routing
!
router isis 1
   net 49.0001.0100.0300.4000.00
   router-id ipv4 10.3.4.0
   advertise passive-only
   authentication mode md5
   authentication key 7 btRsZaMBerY=
   !
   address-family ipv4 unicast
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
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Ethernet10
   description Spine_1:Eth3
   no switchport
   ip address 10.2.6.4/31
   isis enable 1
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 btRsZaMBerY=
!
interface Loopback0
   ip address 10.3.8.0/32
!
interface Loopback100
   ip address 10.3.9.0/32
   isis enable 1
   isis passive
!
interface Management1
!
interface Vlan10
   description App1
   ip address 10.3.10.254/24
   isis enable 1
   isis passive
!
ip routing
!
router isis 1
   net 49.0002.0100.0300.8000.00
   advertise passive-only
   authentication mode md5
   authentication key 7 btRsZaMBerY=
   !
   address-family ipv4 unicast
!
```
### Проверка работы протокола ISIS

1. Проверим, поднялись ли соедские отношения на примере Spine_0
````
Spine_0#show isis neighbors 
 
Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id          
1         default  Leaf_0           L2   Ethernet1          P2P               UP    24          19                  
1         default  Leaf_1           L2   Ethernet2          P2P               UP    24          19                  
1         default  Leaf_2           L2   Ethernet3          P2P               UP    23          19           
````
Как видим, соседство в состоянии UP для всех 3х Leaf. Соседства со Spine_1 быть не должно, так как нет прямых линков до него

2. Проверим ISIS анонсы
````
Spine_0#show isis database 

IS-IS Instance: 1 VRF: default
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Spine_0.00-00               691  24208   700    124 L2 <>
    Spine_1.00-00               674  50809   525    124 L2 <>
    Leaf_0.00-00                691  23165   783    135 L2 <>
    Leaf_1.00-00                663   3174   910    135 L2 <>
    Leaf_2.00-00                105   5614   825    135 L2 <>

````
Все Spine у нас роутеры Level-2. Как следствие, в базе ISIS мы видим базу только Level-2

3. Взглянем на таблицу маршрутизации Spine_0
````
Spine_0#show ip route isis

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 I L2     10.3.1.0/32 [115/20] via 10.2.2.0, Ethernet1
 I L2     10.3.2.0/24 [115/20] via 10.2.2.0, Ethernet1
 I L2     10.3.5.0/32 [115/20] via 10.2.2.2, Ethernet2
 I L2     10.3.6.0/24 [115/20] via 10.2.2.2, Ethernet2
 I L2     10.3.9.0/32 [115/20] via 10.2.2.4, Ethernet3
 I L2     10.3.10.0/24 [115/20] via 10.2.2.4, Ethernet3
 `````
 Мы видим, что p2p подсети отстутсвуют в таблице. На данный момент необходимости в них нет. Это повышает безопасность сети а так же снижает нагрузку на оборудование

4. Что же у нас на уровне Leaf на примере Leaf_0?

`````
Leaf_0(config-router-isis)#show isis database 

IS-IS Instance: 1 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Leaf_0.00-00                688  41026  1037    111 L2 <DefaultAtt>
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Spine_0.00-00               692  57070  1078    124 L2 <>
    Spine_1.00-00               675  55049   996    124 L2 <>
    Leaf_0.00-00                691  23165   447    135 L2 <>
    Leaf_1.00-00                663   3174   573    135 L2 <>
    Leaf_2.00-00                105   5614   488    135 L2 <>
`````

Тут уже видим две базы ISIS: база уровня 2 и база уровня 1.

5. Маршруты на уровне Leaf.
````
Leaf_0(config-router-isis)# show ip route isis

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

 I L2     10.3.5.0/32 [115/30] via 10.2.2.1, Ethernet9
                               via 10.2.6.1, Ethernet10
 I L2     10.3.6.0/24 [115/30] via 10.2.2.1, Ethernet9
                               via 10.2.6.1, Ethernet10
 I L2     10.3.9.0/32 [115/30] via 10.2.2.1, Ethernet9
                               via 10.2.6.1, Ethernet10
 I L2     10.3.10.0/24 [115/30] via 10.2.2.1, Ethernet9
                                via 10.2.6.1, Ethernet10
````
Тут уже видим, что имеется два равнозначных маршрута до Looback100 и SVI интефейсов остальных Leaf, что позволит при включении ecmp использовать их одновременно.

6. Ну, и проверим связность клиентов на примере Client_1
````
VPCS> ping 10.3.6.1

84 bytes from 10.3.6.1 icmp_seq=1 ttl=61 time=83.241 ms
84 bytes from 10.3.6.1 icmp_seq=2 ttl=61 time=69.597 ms
84 bytes from 10.3.6.1 icmp_seq=3 ttl=61 time=49.385 ms
84 bytes from 10.3.6.1 icmp_seq=4 ttl=61 time=65.990 ms
^C
VPCS> ping 10.3.10.1

84 bytes from 10.3.10.1 icmp_seq=1 ttl=61 time=109.887 ms
84 bytes from 10.3.10.1 icmp_seq=2 ttl=61 time=40.014 ms
84 bytes from 10.3.10.1 icmp_seq=3 ttl=61 time=41.454 ms
84 bytes from 10.3.10.1 icmp_seq=4 ttl=61 time=43.891 ms
^C
VPCS> ping 10.3.10.2 

84 bytes from 10.3.10.2 icmp_seq=1 ttl=61 time=61.761 ms
84 bytes from 10.3.10.2 icmp_seq=2 ttl=61 time=44.030 ms
84 bytes from 10.3.10.2 icmp_seq=3 ttl=61 time=40.082 ms
84 bytes from 10.3.10.2 icmp_seq=4 ttl=61 time=46.095 ms
84 bytes from 10.3.10.2 icmp_seq=5 ttl=61 time=45.455 ms

VPCS> 
````
Несмотря на то, что мы отключили анонс p2p линоков, это не мешает конечному оборудованию взаимодействовать друг с другом.
