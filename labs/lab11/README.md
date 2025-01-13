# VxLAN. EVPN L2

### Цели
- Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами


### Топология сети
![BGP-L2VPN.png](/labs/lab11/BGP-L2VPN.png)

### Описание топологии
1. Для Overlay и Underlay сети используется протокол iBGP
2. Все интерфейсы всех комутаторов всех уровней находятся в AS 64512
3. Router ID всех коммутаторов является IP адрес своего Loopback 0
4. Подсети Loopback0 интерфейсов не анонсируются на всех уровнях
5. Интерфейсы >= Loopback100 анонсируются как на уровнях Spine и Leaf
6. На всех p2p линках с подсетью /31 используется функция "netwok type point-to-point"
7. Для всех анонсируемых суммированных маршрутов добавляем в таблицу маршрутизации через интерфейс Null0
8. Spine0 и Spine1 выполняют роль Route-Reflector для Underlay и Overlay сетей
9. BGP соседства для Underlay сети устанавливаются на уровне физических интерфейсов
10. BGP соседства для Overylay сети устанавливаются на уровне интерфейсой >=Loopback100
11. Для Overlay сети используется Route Distiguisher type 1 и соостоит из Loopback0 IP адреса и номера VLAN, который нужно прокинуть через VxLAN туннель
12. VNI равен 1xxxx, где xxxx - номер VLAN, который нужно пробросить через VxLAN туннель


### Адреса устройств

#### Адреса коммутатора Spine 0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.0.0|32|Loopback 0|RID,RD
10.2.1.0|32|Loopback 100|Reserved
10.2.2.1|31|Eth 1|p2p leaf 0
10.2.2.3|31|Eth 2|p2p leaf 1
10.2.2.5|31|Eth 3|p2p leaf 2

#### Адреса коммутатора Spine 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.4.0|32|Loopback 0|RID,RD
10.2.5.0|32|Loopback 100|Reserved
10.2.6.1|31|Eth 1|p2p leaf 0
10.2.6.3|31|Eth 2|p2p leaf 1
10.2.6.5|31|Eth 3|p2p leaf 2


#### Адреса коммутатора Leaf 0
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.0.0|32|Loopback 0|10.3.0.0|RID,RD
10.3.1.0|32|Loopback 100|10.3.0.0|Lo overlay
10.2.2.0|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.0|31|Eth 10|0.0.0.0|p2p spine 1

#### Адреса коммутатора Leaf 1
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.4.0|32|Loopback 0|10.3.4.0|RID,RD
10.3.5.0|32|Loopback 100|10.3.4.0|Lo overlay
10.2.2.2|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.2|31|Eth 10|0.0.0.0|p2p spine 1

#### Адреса коммутатора Leaf 2
IP|маска|Интерфейс|OSPF Area|Назначение
|---|---|---|---|---|
10.3.8.0|32|Loopback 0|10.3.8.0|RID,RD
10.3.9.0|32|Loopback 100|10.3.8.0|Lo overlay
10.2.2.4|31|Eth 9|0.0.0.0|p2p spine 0
10.2.6.4|31|Eth 10|0.0.0.0|p2p spine 1

#### Данные подключения клиентов
Клиент|Leaf sw|Port|IP|Gateway|Mask|VLAN
|---|---|---|---|---|---|---|
Client1|Leaf_0|1|192.168.1.1|192.168.1.254|24|10
Client2|Leaf_1|1|192.168.2.2|192.168.2.254|24|20
Client3|Leaf_2|1|192.168.1.3|192.168.1.254|24|10
Clietn4|Leaf_2|2|192.168.2.4|192.168.2.254|24|20


### Подготовка стенда

#### [Настройка Spine_0](Spine_0.cfg)

```
router bgp 64512
   router-id 10.2.0.0
   maximum-paths 4 ecmp 64
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback100
   neighbor evpn route-reflector-client
   neighbor evpn password 7 87NCY2eLqQQ=
   neighbor evpn send-community extended
   neighbor underlay peer group
   neighbor underlay remote-as 64512
   neighbor underlay bfd
   neighbor underlay route-reflector-client
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.2.0 peer group underlay
   neighbor 10.2.2.0 description Leaf_0
   neighbor 10.2.2.2 peer group underlay
   neighbor 10.2.2.2 description Leaf_1
   neighbor 10.2.2.4 peer group underlay
   neighbor 10.2.2.4 description Leaf_2
   neighbor 10.3.1.0 peer group evpn
   neighbor 10.3.1.0 description Leaf_0
   neighbor 10.3.5.0 peer group evpn
   neighbor 10.3.5.0 description Leaf_1
   neighbor 10.3.9.0 peer group evpn
   neighbor 10.3.9.0 description Leaf_2
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.2.1.0/24
      network 10.2.2.0/24
!



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
ip routing
!
ip route 10.2.6.0/24 Null0
!
router bgp 64512
   router-id 10.2.4.0
   neighbor 10.2.6.0 remote-as 64512
   neighbor 10.2.6.0 bfd
   neighbor 10.2.6.0 description Leaf0
   neighbor 10.2.6.0 route-reflector-client
   neighbor 10.2.6.0 password 7 E3IUT8qCXKI=
   neighbor 10.2.6.2 remote-as 64512
   neighbor 10.2.6.2 bfd
   neighbor 10.2.6.2 description Leaf1
   neighbor 10.2.6.2 route-reflector-client
   neighbor 10.2.6.2 password 7 chfxYS8rP44=
   neighbor 10.2.6.4 remote-as 64512
   neighbor 10.2.6.4 bfd
   neighbor 10.2.6.4 description Leaf2
   neighbor 10.2.6.4 route-reflector-client
   neighbor 10.2.6.4 password 7 QTE2ovjU2pY=
   network 10.2.6.0/24
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
!
interface Vlan10
   description App1
   ip address 10.3.2.254/24
!
ip routing
!
ip route 10.3.1.0/24 Null0
ip route 10.3.2.0/24 Null0
!
router bgp 64512
   router-id 10.3.0.0
   maximum-paths 2
   neighbor 10.2.2.1 remote-as 64512
   neighbor 10.2.2.1 bfd
   neighbor 10.2.2.1 description Spine0
   neighbor 10.2.2.1 password 7 XWFo5YHkqfI=
   neighbor 10.2.6.1 remote-as 64512
   neighbor 10.2.6.1 bfd
   neighbor 10.2.6.1 description Spine1
   neighbor 10.2.6.1 password 7 E3IUT8qCXKI=
   network 10.3.1.0/24
   network 10.3.2.0/24
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
interface Management1
!
interface Vlan10
   description App1
   ip address 10.3.6.254/24
!
ip routing
!
ip route 10.3.5.0/24 Null0
ip route 10.3.6.0/24 Null0
!
router bgp 64512
   router-id 10.3.4.0
   maximum-paths 2
   neighbor 10.2.2.3 remote-as 64512
   neighbor 10.2.2.3 bfd
   neighbor 10.2.2.3 password 7 CDMhtdkY700=
   neighbor 10.2.6.3 remote-as 64512
   neighbor 10.2.6.3 bfd
   neighbor 10.2.6.3 password 7 chfxYS8rP44=
   network 10.3.5.0/24
   network 10.3.6.0/24
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
!
ip routing
!
ip route 10.3.9.0/24 Null0
ip route 10.3.10.0/24 Null0
!
router bgp 64512
   maximum-paths 2
   neighbor 10.2.2.5 remote-as 64512
   neighbor 10.2.2.5 bfd
   neighbor 10.2.2.5 description Spine0
   neighbor 10.2.2.5 password 7 8jY+bQI+EZc=
   neighbor 10.2.6.5 remote-as 64512
   neighbor 10.2.6.5 bfd
   neighbor 10.2.6.5 description Spine1
   neighbor 10.2.6.5 password 7 QTE2ovjU2pY=
   network 10.3.9.0/24
   network 10.3.10.0/24
!
```
### Проверка работы протокола BGP

1. Проверим, поднялись ли соедские отношения на примере Spine_0
````
Spine_0#show ip bgp summary 
BGP summary information for VRF default
Router identifier 10.2.0.0, local AS number 64512
Neighbor Status Codes: m - Under maintenance
  Description              Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  Leaf0                    10.2.2.0         4  64512          35660     35677    0    0   24d18h Estab   2      2
  Leaf1                    10.2.2.2         4  64512          35622     35642    0    0 05:30:17 Estab   2      2
  Leaf2                    10.2.2.4         4  64512          34242     34261    0    0   23d18h Estab   2      2
        
````
Как видим, соседство в состоянии Established для всех 3х Leaf. От каждого Leaf получено по два префикса и каждому Leaf отправлено 

2. Взглянем на таблицу маршрутизации Spine_0
````
Spine_0#show ip route bgp

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

 B I      10.3.1.0/24 [200/0] via 10.2.2.0, Ethernet1
 B I      10.3.2.0/24 [200/0] via 10.2.2.0, Ethernet1
 B I      10.3.5.0/24 [200/0] via 10.2.2.2, Ethernet2
 B I      10.3.6.0/24 [200/0] via 10.2.2.2, Ethernet2
 B I      10.3.9.0/24 [200/0] via 10.2.2.4, Ethernet3
 B I      10.3.10.0/24 [200/0] via 10.2.2.4, Ethernet3

 `````
 Мы видим, что присутствуют подсети Loopback100 и Clinet

3. Что же у нас на уровне Leaf на примере Leaf_0?

`````
Leaf_0#show ip bgp summary 
BGP summary information for VRF default
Router identifier 10.3.0.0, local AS number 64512
Neighbor Status Codes: m - Under maintenance
  Description              Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  Spine0                   10.2.2.1         4  64512          35766     35749    0    0   24d19h Estab   5      5
  Spine1                   10.2.6.1         4  64512          35688     35675    0    0   24d18h Estab   5      5
`````

Мы видим, что BGP сессия установлена с двумя Spine, получено и установлено по 5 префиксов с каждого Leaf.

4. Маршруты на уровне Leaf.
````
Leaf_0#show ip route bgp 

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

 B I      10.2.2.0/24 [200/0] via 10.2.2.1, Ethernet9
 B I      10.2.6.0/24 [200/0] via 10.2.6.1, Ethernet10
 B I      10.3.5.0/24 [200/0] via 10.2.2.1, Ethernet9
                              via 10.2.6.1, Ethernet10
 B I      10.3.6.0/24 [200/0] via 10.2.2.1, Ethernet9
                              via 10.2.6.1, Ethernet10
 B I      10.3.9.0/24 [200/0] via 10.2.2.1, Ethernet9
                              via 10.2.6.1, Ethernet10
 B I      10.3.10.0/24 [200/0] via 10.2.2.1, Ethernet9
                               via 10.2.6.1, Ethernet10


````
Тут уже видим, что имеется два равнозначных маршрута до Looback100 и SVI интефейсов остальных Leaf, что позволит при включении ecmp использовать их одновременно. Более того, присуствуют маршруты p2p линков. Причина этих маршрутов - iBGP не меняет next-hop аттрибут при передачи маршрута и при отсутствии p2p маршрутов, префиксы до Loopback100 и сервисных подсетей не будут внесены в основнуйю таблицу маршрутизации.

````
Leaf_0#show ip bgp neighbors 10.2.2.1 received-routes 
BGP routing table information for VRF default
Router identifier 10.3.0.0, local AS number 64512
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * >     10.2.2.0/24            10.2.2.1              -       100     -       ?
 *  ec   10.3.5.0/24            10.2.2.2              -       100     -       ? Or-ID: 10.3.4.0 C-LST: 10.2.0.0
 *  ec   10.3.6.0/24            10.2.2.2              -       100     -       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0
 *  ec   10.3.9.0/24            10.2.2.4              -       100     -       ? Or-ID: 10.3.9.0 C-LST: 10.2.0.0
 *  ec   10.3.10.0/24           10.2.2.4              -       100     -       i Or-ID: 10.3.9.0 C-LST: 10.2.0.0

````
Как видим Next-Hop для префиксов, анонсированных другими Leaf, является их p2p IP адрес

5. Ну, и проверим связность клиентов на примере Client_1
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
