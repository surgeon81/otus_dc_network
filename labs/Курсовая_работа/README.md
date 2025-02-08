# VxLAN. EVPN L3

### Цели
- Реализовать маршрутизацию между "клиентами" через вншенее устройство посредством EVPN route-type 5



### Топология сети
![BGP-Routing](/labs/Курсовая_работа/image.png)

### Описание топологии
1. Для Overlay и Underlay сети используется протокол iBGP
2. Все интерфейсы всех комутаторов всех уровней находятся в AS 64512
3. Router ID всех коммутаторов является IP адрес своего Loopback 0
4. Подсети Loopback0 интерфейсов не анонсируются на всех уровнях
5. Интерфейсы >= Loopback100 анонсируются как на уровнях Spine и Leaf и используются для VXLAN туннеленй
6. На всех p2p линках с подсетью /31 используется функция "netwok type point-to-point"
7. Для всех анонсируемых суммированных маршрутов добавляем в таблицу маршрутизации через интерфейс Null0
8. Spine0 и Spine1 выполняют роль Route-Reflector для Underlay и Overlay сетей
9. BGP соседства для Underlay сети устанавливаются на уровне физических интерфейсов
10. BGP соседства для Overylay сети устанавливаются на уровне интерфейсой >=Loopback100
11. Для Overlay сети используется Route Distiguisher type 1 и соостоит из Loopback0 IP адреса и номера VLAN, который нужно прокинуть через VxLAN туннель
12. VNI равен - номеру клиентского VLAN
13. Для маршрутизации используется Symmtetric IRB
14. Для снижения ARP запросов используется anycast gateway c виртуальным MAC
15. Для большей наглядности и уменьшения нагрузки анонсировавание MAC-IP типов маршрутов не реализовано.
16. В качестве протокола маршрутизации между внешним ротуром (Router) испльзуется OSPF
17. С целью экономии tcam, в фабрику со стороны Router будут анонсироваться только маршрурт по умолчанию
18. Каждый клиент а так же линки между Leaf_2 м Router находzтся в своем отдельном VLAN и VRF
19. Клиенты 10 и 30 относятся к одной органиции а Клиенты 20 и 40 к другой. Клиенты одной организации имеют доступ друг другу в пределах фабрики. Связь между организациями осуществляется только за пределами фабрики через Router


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
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.0.0|32|Loopback 0|RID,RD
10.3.1.0|32|Loopback 100|Lo overlay
10.2.2.0|31|Eth 9|p2p spine 0
10.2.6.0|31|Eth 10|p2p spine 1
192.168.10.254|24|VLAN 10|GW Client_10
192.168.20.254|24|VLAN 20|GW Client_20

#### Адреса коммутатора Leaf 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.4.0|32|Loopback 0|RID,RD
10.3.5.0|32|Loopback 100|Lo overlay
10.2.2.2|31|Eth 9|p2p spine 0
10.2.6.2|31|Eth 10|p2p spine 1
192.168.30.254|24|VLAN 30|GW Client_30
192.168.40.254|24|VLAN 40|GW Client_40

#### Адреса коммутатора Leaf 2 (Border Leaf)
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.8.0|32|Loopback 0|RID,RD
10.3.9.0|32|Loopback 100|Lo overlay
10.2.2.4|31|Eth 9|p2p spine 0
10.2.6.4|31|Eth 10|p2p spine 1
192.168.100.254|24|VLAN 100|p2p Leaf_2, Client 10,30
192.168.200.254|24|VLAN 200|p2p Leaf_2, Client 20,40

#### Адреса коммутатора Router
IP|маска|Интерфейс|Назначение
|---|---|---|---|
192.168.100.1|24|VLAN 100|p2p Leaf_2
192.168.200.1|24|VLAN 200|p2p Leaf_2

#### Данные подключения клиентов
Клиент|Leaf sw|Port|IP|Gateway|Mask|VLAN
|---|---|---|---|---|---|---|
Client10|Leaf_0|1|192.168.10.1|192.168.10.254|24|10
Client20|Leaf_0|1|192.168.20.1|192.168.20.254|24|20
Client30|Leaf_2|1|192.168.30.1|192.168.30.254|24|30
Clietn40|Leaf_2|2|192.168.40.1|192.168.50.254|24|40


### Подготовка стенда

Пропустим уровень Spine, так как там ничего особенного нет и конфигурация не отличается от других лабораторных
###### [Конфигурация Spine_0](Spine_0.cfg)
###### [Конфигурация Spine_1](Spine_1.cfg)


#### [Конфигурация Leaf_0](Leaf_0.cfg)

```
vlan 10,20
!
vrf instance Cust_10
!
vrf instance Cust_20
!
interface Ethernet1
   description Client_10:eth0
   switchport access vlan 10
!
interface Ethernet2
   description Client_20:eth0
   switchport access vlan 20
!
interface Management1
!
interface Vlan10
   vrf Cust_10
   ip address virtual 192.168.10.254/24
!
interface Vlan20
   vrf Cust_20
   ip address virtual 192.168.20.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vrf Cust_10 vni 10
   vxlan vrf Cust_20 vni 20
!
ip routing
ip routing vrf Cust_10
ip routing vrf Cust_20
!
ip route 10.3.1.0/24 Null0
ip route 10.3.2.0/24 Null0
!
router bgp 64512
   router-id 10.3.0.0
   maximum-paths 4 ecmp 64
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback100
   neighbor evpn password 7 87NCY2eLqQQ=
   neighbor evpn send-community extended
   neighbor underlay peer group
   neighbor underlay remote-as 64512
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.1.0 description Spine_0
   neighbor 10.2.2.1 peer group underlay
   neighbor 10.2.2.1 remote-as 64512
   neighbor 10.2.2.1 description Spine_0
   neighbor 10.2.2.1 password 7 XWFo5YHkqfI=
   neighbor 10.2.5.0 peer group evpn
   neighbor 10.2.5.0 description Spine_1
   neighbor 10.2.6.1 peer group underlay
   neighbor 10.2.6.1 remote-as 64512
   neighbor 10.2.6.1 description Spine_1
   neighbor 10.2.6.1 password 7 E3IUT8qCXKI=
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.1.0/24
   !
   vrf Cust_10
      rd 10.3.0.0:10
      route-target import evpn 64512:1030
      route-target export evpn 64512:1030
      redistribute connected
   !
   vrf Cust_20
      rd 10.3.0.0:20
      route-target import evpn 64512:2040
      route-target export evpn 64512:2040
      redistribute connected
!
end

```

 #### [Настройка Leaf_1](Leaf_1.cfg)

```
vlan 30,40
!
vrf instance Cust_30
!
vrf instance Cust_40
!
interface Ethernet1
   description Client_30:eth0
   switchport access vlan 30
!
interface Ethernet2
   description Client_40:eth0
   switchport access vlan 40
!
interface Vlan30
   vrf Cust_30
   ip address virtual 192.168.30.254/24
!
interface Vlan40
   vrf Cust_40
   ip address virtual 192.168.40.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vrf Cust_30 vni 30
   vxlan vrf Cust_40 vni 40
!
ip routing
ip routing vrf Cust_30
ip routing vrf Cust_40
!
ip route 10.3.5.0/24 Null0
ip route 10.3.6.0/24 Null0
!
router bgp 64512
   router-id 10.3.4.0
   maximum-paths 4 ecmp 64
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback100
   neighbor evpn password 7 87NCY2eLqQQ=
   neighbor evpn send-community extended
   neighbor underlay peer group
   neighbor underlay remote-as 64512
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.1.0 description Spine_0
   neighbor 10.2.2.3 peer group underlay
   neighbor 10.2.2.3 description Spine_0
   neighbor 10.2.5.0 peer group evpn
   neighbor 10.2.5.0 description Spine_1
   neighbor 10.2.6.3 peer group underlay
   neighbor 10.2.6.3 description Spine_1
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.5.0/24
   !
   vrf Cust_30
      rd 10.3.4.0:30
      route-target import evpn 64512:1030
      route-target export evpn 64512:1030
      redistribute connected
   !
   vrf Cust_40
      rd 10.3.4.0:40
      route-target import evpn 64512:2040
      route-target export evpn 64512:2040
      redistribute connected
!
end

```

 #### [Настройка Leaf_2](Leaf_2.cfg)

```
vrf instance VLAN100
!
vrf instance VLAN200
!
interface Ethernet1
   description Router1:Eth1
   switchport trunk allowed vlan 100,200
   switchport mode trunk
!
interface Vlan100
   vrf VLAN100
   ip address 192.168.100.254/24
!
interface Vlan200
   vrf VLAN200
   ip address 192.168.200.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vrf VLAN100 vni 100
   vxlan vrf VLAN200 vni 200
!
ip routing
ip routing vrf VLAN100
ip routing vrf VLAN200
!
ip route 10.3.9.0/24 Null0
ip route 10.3.10.0/24 Null0
!
route-map Default_Router_RM permit 10
   match tag 100200
!
router bgp 64512
   router-id 10.3.8.0
   maximum-paths 4 ecmp 64
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback100
   neighbor evpn password 7 87NCY2eLqQQ=
   neighbor evpn send-community extended
   neighbor underlay peer group
   neighbor underlay remote-as 64512
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.1.0 description Spine_0
   neighbor 10.2.2.5 peer group underlay
   neighbor 10.2.2.5 description Spine_0
   neighbor 10.2.5.0 peer group evpn
   neighbor 10.2.5.0 description Spine_1
   neighbor 10.2.6.5 peer group underlay
   neighbor 10.2.6.5 description Spine_1
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.9.0/24
   !
   vrf VLALAN100
   !
   vrf VLAN100
      rd 10.3.8.0:100
      route-target import evpn 64512:1030
      route-target export evpn 64512:1030
      redistribute ospf match external route-map Default_Router_RM
   !
   vrf VLAN200
      rd 10.3.8.0:200
      route-target import evpn 64512:2040
      route-target export evpn 64512:2040
      redistribute ospf match external route-map Default_Router_RM
!
router ospf 100 vrf VLAN100
   router-id 192.168.100.254
   redistribute bgp
   network 192.168.100.254/32 area 0.0.0.0
   max-lsa 12000
!
router ospf 200 vrf VLAN200
   router-id 192.168.200.254
   redistribute bgp
   network 192.168.200.254/32 area 0.0.0.0
   max-lsa 12000
!
end
```
 #### [Настройка Router](Router.cfg)
 ```
vlan 100,200
!
interface Ethernet1
   switchport trunk allowed vlan 100,200
   switchport mode trunk
!
interface Vlan100
   ip address 192.168.100.1/24
!
interface Vlan200
   ip address 192.168.200.1/24
!
ip access-list standard Default_route
   10 permit any
!
ip routing
!
ip route 0.0.0.0/0 Null0
!
route-map Default_route_RM permit 10
   match ip address access-list Default_route
   set tag 100200
!
router ospf 1
   network 192.168.100.1/32 area 0.0.0.0
   network 192.168.200.1/32 area 0.0.0.0
   max-lsa 12000
   default-information originate always route-map Default_route_RM
!
end

 ```

### Проверка работоспособности EVPN 

1. Для большей наглядности, на время отключим Router 1 и проверим наличие маршрутов EVPN а так же связности клиентов на примере Leaf_0
````
Leaf_0#show bgp evpn 
BGP routing table information for VRF default
Router identifier 10.3.0.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.3.0.0:10 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >      RD: 10.3.0.0:20 ip-prefix 192.168.20.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.3.4.0:30 ip-prefix 192.168.30.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 
 *  ec    RD: 10.3.4.0:30 ip-prefix 192.168.30.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.4.0 
 * >Ec    RD: 10.3.4.0:40 ip-prefix 192.168.40.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.4.0 
 *  ec    RD: 10.3.4.0:40 ip-prefix 192.168.40.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 
        
````
Мы видим Route-Type 5 маршруты для подсетей каждого клиента

2. Взглянем на таблицу маршрутизации клиентских VRF
````
Leaf_0#show ip route vrf Cust_10

VRF: Cust_10
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

Gateway of last resort is not set

 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.30.0/24 [200/0] via VTEP 10.3.5.0 VNI 30 router-mac 50:00:00:03:37:66 local-interface Vxlan1

Leaf_0#show ip route vrf Cust_20

VRF: Cust_20
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

Gateway of last resort is not set

 C        192.168.20.0/24 is directly connected, Vlan20
 B I      192.168.40.0/24 [200/0] via VTEP 10.3.5.0 VNI 40 router-mac 50:00:00:03:37:66 local-interface Vxlan1

Leaf_0#

`````
 Мы видим два маршрута в каждом VRF: непосредственно адрес сети клиента, чей VRF находится на Leaf_0 а так же маршруты, полученные по BGP от Leaf_1

3. Проверим связность клинетов на примере Client_10

`````
Client_10> ping 192.168.30.1

84 bytes from 192.168.30.1 icmp_seq=1 ttl=62 time=177.702 ms
84 bytes from 192.168.30.1 icmp_seq=2 ttl=62 time=51.706 ms
84 bytes from 192.168.30.1 icmp_seq=3 ttl=62 time=60.399 ms
84 bytes from 192.168.30.1 icmp_seq=4 ttl=62 time=59.961 ms
84 bytes from 192.168.30.1 icmp_seq=5 ttl=62 time=65.395 ms

Client_10> ping 192.168.20.1

*192.168.10.254 icmp_seq=1 ttl=64 time=16.948 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=2 ttl=64 time=12.179 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=3 ttl=64 time=9.198 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=4 ttl=64 time=11.462 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=5 ttl=64 time=11.869 ms (ICMP type:3, code:0, Destination network unreachable)

Client_10> ping 192.168.40.1

*192.168.10.254 icmp_seq=1 ttl=64 time=8.676 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=2 ttl=64 time=17.957 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=3 ttl=64 time=8.126 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=4 ttl=64 time=7.823 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.254 icmp_seq=5 ttl=64 time=9.746 ms (ICMP type:3, code:0, Destination network unreachable)

`````

Мы видим, что связность в пределах фабрики есть внтури одной организации и нет между.

4. Включим Router и проверим обновленную таблицу evpn маршрутизации
````
Leaf_0#show bgp evpn 
BGP routing table information for VRF default
Router identifier 10.3.0.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.3.8.0:100 ip-prefix 0.0.0.0/0
                                 10.3.9.0              -       100     0       i Or-ID: 10.3.8.0 C-LST: 10.2.4.0 
 *  ec    RD: 10.3.8.0:100 ip-prefix 0.0.0.0/0
                                 10.3.9.0              -       100     0       i Or-ID: 10.3.8.0 C-LST: 10.2.0.0 
 * >Ec    RD: 10.3.8.0:200 ip-prefix 0.0.0.0/0
                                 10.3.9.0              -       100     0       i Or-ID: 10.3.8.0 C-LST: 10.2.4.0 
 *  ec    RD: 10.3.8.0:200 ip-prefix 0.0.0.0/0
                                 10.3.9.0              -       100     0       i Or-ID: 10.3.8.0 C-LST: 10.2.0.0 
 * >      RD: 10.3.0.0:10 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >      RD: 10.3.0.0:20 ip-prefix 192.168.20.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.3.4.0:30 ip-prefix 192.168.30.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 
 *  ec    RD: 10.3.4.0:30 ip-prefix 192.168.30.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.4.0 
 * >Ec    RD: 10.3.4.0:40 ip-prefix 192.168.40.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.4.0 
 *  ec    RD: 10.3.4.0:40 ip-prefix 192.168.40.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 

````
Что-же у нас с таблицей маршрутизации клиентский vrf?


````
Leaf_0#show ip route vrf Cust_10

VRF: Cust_10
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

Gateway of last resort:
 B I      0.0.0.0/0 [200/0] via VTEP 10.3.9.0 VNI 100 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1

 C        192.168.10.0/24 is directly connected, Vlan10
 B I      192.168.30.0/24 [200/0] via VTEP 10.3.5.0 VNI 30 router-mac 50:00:00:03:37:66 local-interface Vxlan1

Leaf_0#show ip route vrf Cust_20

VRF: Cust_20
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

Gateway of last resort:
 B I      0.0.0.0/0 [200/0] via VTEP 10.3.9.0 VNI 200 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1

 C        192.168.20.0/24 is directly connected, Vlan20
 B I      192.168.40.0/24 [200/0] via VTEP 10.3.5.0 VNI 40 router-mac 50:00:00:03:37:66 local-interface Vxlan1
````
Как видим мы получам дополнительный Route type 5 - 0.0.0.0/0. Благодаря route-map для редистрибьюции из OSPF в BGP мы не видим префиков сетей между Leaf_2 и Router, а так же не видим реанонс клиентских префиксов обратно в фабрику через OSPF.
Таблица маршрутизации VRF так же ограничена маршрутом по умолчанию с префиксами одной организации

6. ~~Попробуем теперь с этой всей фигней взлететь~~ Проверим связность
````
Client_10> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=59 time=170.600 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=59 time=134.526 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=59 time=124.657 ms
84 bytes from 192.168.20.1 icmp_seq=4 ttl=59 time=211.685 ms
84 bytes from 192.168.20.1 icmp_seq=5 ttl=59 time=129.843 ms

Client_10> ping 192.168.30.1

84 bytes from 192.168.30.1 icmp_seq=1 ttl=62 time=154.360 ms
84 bytes from 192.168.30.1 icmp_seq=2 ttl=62 time=54.076 ms
84 bytes from 192.168.30.1 icmp_seq=3 ttl=62 time=48.118 ms
84 bytes from 192.168.30.1 icmp_seq=4 ttl=62 time=77.460 ms
84 bytes from 192.168.30.1 icmp_seq=5 ttl=62 time=69.877 ms

Client_10> ping 192.168.40.1

84 bytes from 192.168.40.1 icmp_seq=1 ttl=59 time=178.442 ms
84 bytes from 192.168.40.1 icmp_seq=2 ttl=59 time=150.568 ms
84 bytes from 192.168.40.1 icmp_seq=3 ttl=59 time=245.449 ms
84 bytes from 192.168.40.1 icmp_seq=4 ttl=59 time=141.128 ms
84 bytes from 192.168.40.1 icmp_seq=5 ttl=59 time=154.207 ms
...

