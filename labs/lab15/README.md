# VxLAN. EVPN L3

### Цели
- Реализовать маршрутизацию между "клиентами" через вншенее устройство посредством EVPN route-type 5



### Топология сети
![BGP-Routing](/labs/lab15/image.png)

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
17. Каждый клиент а так же линки между Leaf_2 м Router находzтся в своем отдельном VLAN и VRF
18. Клиенты 10 и 30 относятся к одной органиции а Клиенты 20 и 40 к другой. Клиенты одной организации имеют доступ друг другу в пределах фабрики. Связь между организациями осуществляется только за пределами фабрики через Router


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
vlan 100,200
!
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
   vrf VLAN100
      rd 10.3.8.0:100
      route-target import evpn 64512:1030
      route-target export evpn 64512:1030
      redistribute connected
      redistribute ospf
      redistribute ospf match external
   !
   vrf VLAN200
      rd 10.3.8.0:10020
      route-target import evpn 64512:2040
      route-target export evpn 64512:2040
      redistribute connected
      redistribute ospf
      redistribute ospf match external
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
### Проверка работоспособности маршрутизации между протокола BGP

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

3. Проверим связность клинетов

`````
Leaf_2# show bgp summary 
BGP summary information for VRF default
Router identifier 10.3.8.0, local AS number 64512
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.2.1.0       64512 Established   IPv4 Unicast            Negotiated              4          4
10.2.1.0       64512 Established   L2VPN EVPN              Negotiated              3          3
10.2.2.5       64512 Established   IPv4 Unicast            Negotiated              4          4
10.2.5.0       64512 Established   IPv4 Unicast            Negotiated              4          4
10.2.5.0       64512 Established   L2VPN EVPN              Negotiated              3          3
10.2.6.5       64512 Established   IPv4 Unicast            Negotiated              4          4

`````

Мы видим, что BGP сессии установлены с двумя Spine как для передачи IPv4 маршрутов так и для L2VPN информации.

4. Маршруты на уровне Leaf не ссильно отличаются от уровня Spine. Но у нас еще появляются маршруты 5го типа - Route Type 5, для анонсирования подсетей
````
Leaf_2#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.3.8.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.3.0.0:10000 ip-prefix 192.168.1.0/24
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.4.0 
 *  ec    RD: 10.3.0.0:10000 ip-prefix 192.168.1.0/24
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.0.0 
 * >      RD: 10.3.8.0:10000 ip-prefix 192.168.1.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.3.4.0:10000 ip-prefix 192.168.2.0/24
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 
 *  ec    RD: 10.3.4.0:10000 ip-prefix 192.168.2.0/24                           
 
 * >      RD: 10.3.8.0:10000 ip-prefix 192.168.3.0/24 -         -      0       i  Or-ID: 10.3.4.0 C-LST: 10.2.4.0
                                                           
````
Что-же у нас с мак-таблицей


````
Leaf_2#show mac address-table 
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
   1    0000.8000.0000    STATIC      Cpu
  10    0000.8000.0000    STATIC      Cpu
  10    0050.7966.6806    DYNAMIC     Vx1        1       0:05:00 ago
  10    0050.7966.6808    DYNAMIC     Et1        1       0:05:01 ago
  30    0000.8000.0000    STATIC      Cpu
  30    0050.7966.6809    DYNAMIC     Et2        1       0:04:54 ago
4094    0000.8000.0000    STATIC      Cpu
4094    5000.0003.3766    DYNAMIC     Vx1        1       1 day, 22:18:09 ago
4094    5000.00d5.5dc0    DYNAMIC     Vx1        1       1 day, 22:07:05 ago
````
Как видим мы получам мак-адреса удаленных клиентов через интерфейс VxLAN 1 и локально через Eth порты а так же появились дополнительные мак-адреса в VLAN 4094. Это мак-адреса для работы маршрутизаци

6. Маршруты в VRF
````
Leaf_2#show ip route vrf L3VPN

...

Gateway of last resort is not set

 B I      192.168.1.1/32 [200/0] via VTEP 10.3.1.0 VNI 10000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.1.0/24 is directly connected, Vlan10
 B I      192.168.2.2/32 [200/0] via VTEP 10.3.5.0 VNI 10000 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B I      192.168.2.0/24 [200/0] via VTEP 10.3.5.0 VNI 10000 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        192.168.3.0/24 is directly connected, Vlan30

````
Видим подсети, которые мы импортируем через RT

6. Проверим состояние VxLAN итерфейса
````
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.3.9.0
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is 
    [10, 10010]       [30, 10030]      
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 10000]    
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is 
   [L3VPN, 10000]
  Headend replication flood vtep list is:
    10 10.3.1.0       
  Shared Router MAC is 0000.0000.0000

````
Как мы видим по параметру "Headend replication flood vtep list is" VxLAN туннель у нас установлен с Leaf_0 для VLAN 10 а так же наличия промежуточного VNI для маршрутизации

7. Ну, и проверим связность клиентов на примере Client_1
````
Client_1> ping 192.168.2.2

84 bytes from 192.168.2.2 icmp_seq=1 ttl=62 time=86.222 ms
84 bytes from 192.168.2.2 icmp_seq=2 ttl=62 time=50.935 ms
^C
Client_1> ping 192.168.1.3

84 bytes from 192.168.1.3 icmp_seq=1 ttl=64 time=157.572 ms
84 bytes from 192.168.1.3 icmp_seq=2 ttl=64 time=69.884 ms
^C
Client_1> ping 192.168.3.4

84 bytes from 192.168.3.4 icmp_seq=1 ttl=62 time=94.966 ms
84 bytes from 192.168.3.4 icmp_seq=2 ttl=62 time=82.320 ms
^C


VPCS> 
````
