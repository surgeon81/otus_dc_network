# VxLAN. EVPN L2. Multihoming

### Цели
- Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming 


### Топология сети
![BGP-L3VPN](/labs/lab12/image.png)

### Описание топологии
1. Для Overlay и Underlay сети используется протокол iBGP
2. Все интерфейсы всех комутаторов всех уровней находятся в AS 64512
3. Router ID всех коммутаторов является IP адрес своего Loopback 0
4. Подсети Loopback0 интерфейсов анонсируются на всех уровнях и служат для Overlay BGP
5. Интерфейсы Loopback100 анонсируются только на уровне Leaf и служат конечнымы точками для VxLAN туннеля
6. На всех p2p линках с подсетью /31 используется функция "netwok type point-to-point"
7. Для всех анонсируемых суммированных маршрутов добавляем в таблицу маршрутизации через интерфейс Null0
8. Spine0 и Spine1 выполняют роль Route-Reflector для Underlay и Overlay сетей
9. BGP соседства для Underlay сети устанавливаются на уровне физических интерфейсов
10. BGP соседства для Overylay сети устанавливаются на уровне интерфейсов Loopback0
11. Для Overlay сети используется Route Distiguisher type 1 и соостоит из Loopback 100 IP адреса и номера VLAN, который нужно прокинуть через VxLAN туннель
12. VNI равен 1xxxx, где xxxx - номер VLAN, который нужно пробросить через VxLAN туннель
13. Для настройки VxLAN туннеля используется интерфейс Loopback 100, причем его IP идентичный в пределах MLAG группы
14. Для резервирования на уровне Leaf используется технология MLAG

### Адреса устройств

#### Адреса коммутатора Spine 0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.0.0|32|Loopback 0|RID,RD
10.2.1.0|32|Loopback 100|Reserved
10.2.2.1|31|Eth 1|p2p leaf 0_0
10.2.2.3|31|Eth 2|p2p leaf 0_1
10.2.2.5|31|Eth 3|p2p leaf 1_0
10.2.2.7|31|Eth 4|p2p leaf 1_1

#### Адреса коммутатора Spine 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.4.0|32|Loopback 0|RID,RD
10.2.5.0|32|Loopback 100|Reserved
10.2.6.1|31|Eth 1|p2p leaf 0_0
10.2.6.3|31|Eth 2|p2p leaf 0_1
10.2.6.5|31|Eth 3|p2p leaf 1_0
10.2.6.7|31|Eth 4|p2p leaf 1_1


#### Адреса коммутатора Leaf 0_0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.0.0|32|Loopback 0|RID,RD
10.3.0.128|31|VLAN 4093|Inter-Leaf BGP
10.3.1.0|32|Loopback 100|Lo overlay
10.2.2.0|31|Eth 9|p2p spine 0
10.2.6.0|31|Eth 10|p2p spine 1
192.168.0.0|31|VLAN 4094|MLAG peering

#### Адреса коммутатора Leaf 0_1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.0.1|32|Loopback 0|RID,RD
10.3.0.129|31|VLAN 4093|Inter-Leaf BGP
10.3.1.0|32|Loopback 100|Lo overlay
10.2.2.2|31|Eth 9|p2p spine 0
10.2.6.2|31|Eth 10|p2p spine 1
192.168.0.1|31|VLAN 4094|MLAG peering


#### Адреса коммутатора Leaf 1_0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.4.0|32|Loopback 0|RID,RD
10.3.4.128|31|VLAN 4093|Inter-Leaf BGP
10.3.5.0|32|Loopback 100|Lo overlay
10.2.2.4|31|Eth 9|p2p spine 0
10.2.6.4|31|Eth 10|p2p spine 1
192.168.0.0|31|VLAN 4094|MLAG peering

#### Адреса коммутатора Leaf 1_1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.4.1|32|Loopback 0|RID,RD
10.3.4.129|31|VLAN 4093|Inter-Leaf BGP
10.3.5.0|32|Loopback 100|Lo overlay
10.2.2.6|31|Eth 9|p2p spine 0
10.2.6.6|31|Eth 10|p2p spine 1
192.168.0.1|31|VLAN 4094|MLAG peering

#### Данные подключения клиентов
Клиент|Leaf sw|Port|IP|Gateway|Mask|VLAN
|---|---|---|---|---|---|---|
Client1|SW_1|1|192.168.1.1|192.168.1.254|24|10
Clietn4|SW_2|1|192.168.1.4|192.168.1.254|24|10


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
router bgp 64512
   router-id 10.2.4.0
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
   neighbor 10.2.6.0 peer group underlay
   neighbor 10.2.6.0 description Leaf_0
   neighbor 10.2.6.2 peer group underlay
   neighbor 10.2.6.2 description Leaf_1
   neighbor 10.2.6.4 peer group underlay
   neighbor 10.2.6.4 description Leaf_2
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
      network 10.2.5.0/24
      network 10.2.6.0/24
!
```

#### [Настройка Leaf_0](Leaf_0.cfg)

```
vlan 10
   name App1
!
vrf instance L3VPN
!
interface Ethernet1
   description Client_1:Eth0
   switchport access vlan 10
!
interface Vlan10
   vrf L3VPN
   ip address virtual 192.168.1.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf L3VPN vni 10000
!
ip virtual-router mac-address 00:00:80:00:00:00
!
ip routing
ip routing vrf L3VPN
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
   neighbor underlay bfd
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.1.0 peer group evpn
   neighbor 10.2.1.0 description Spine_0
   neighbor 10.2.2.1 peer group underlay
   neighbor 10.2.2.1 remote-as 64512
   neighbor 10.2.2.1 bfd
   neighbor 10.2.2.1 description Spine_0
   neighbor 10.2.2.1 password 7 XWFo5YHkqfI=
   neighbor 10.2.5.0 peer group evpn
   neighbor 10.2.5.0 description Spine_1
   neighbor 10.2.6.1 peer group underlay
   neighbor 10.2.6.1 remote-as 64512
   neighbor 10.2.6.1 bfd
   neighbor 10.2.6.1 description Spine_1
   neighbor 10.2.6.1 password 7 E3IUT8qCXKI=
   !
   vlan 10
      rd 10.3.0.0:10
      route-target both 64512:10010
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.1.0/24
   !
   vrf L3VPN
      rd 10.3.0.0:10000
      route-target import evpn 64512:10000
      route-target export evpn 64512:10000
      redistribute connected
!
```

 #### [Настройка Leaf_1](Leaf_1.cfg)

```
vlan 20
!
vrf instance L3VPN
!
interface Ethernet1
   description Client_2:Eth0
   switchport access vlan 20
!
interface Vlan20
   vrf L3VPN
   ip address virtual 192.168.2.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vrf L3VPN vni 10000
!
ip virtual-router mac-address 00:00:80:00:00:00
!
ip routing
ip routing vrf L3VPN
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
   neighbor underlay bfd
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
   vlan 20
      rd 10.3.4.0:20
      route-target both 64512:10020
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.5.0/24
   !
   vrf L3VPN
      rd 10.3.4.0:10000
      route-target import evpn 64512:10000
      route-target export evpn 64512:10000
      redistribute connected
!
end
```

 #### [Настройка Leaf_2](Leaf_2.cfg)

```
vlan 10
   name App1
!
vlan 30
!
vrf instance L3VPN
!
interface Ethernet1
   description Client_3:Eth0
   switchport access vlan 10
!
interface Ethernet2
   description Client_4:Eth0
   switchport access vlan 30
!
!
interface Vlan10
   vrf L3VPN
   ip address virtual 192.168.1.254/24
!
interface Vlan30
   vrf L3VPN
   ip address virtual 192.168.3.254/24
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 30 vni 10030
   vxlan vrf L3VPN vni 10000
!
ip virtual-router mac-address 00:00:80:00:00:00
!
ip routing
ip routing vrf L3VPN
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
   neighbor underlay bfd
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
   vlan 10
      rd 10.3.8.0:10
      route-target both 64512:10010
      redistribute learned
   !
   vlan 30
      rd 10.3.8.0:30
      route-target both 64512:10030
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.9.0/24
   !
   vrf L3VPN
      rd 10.3.8.0:10000
      route-target import evpn 64512:10000
      route-target export evpn 64512:10000
      redistribute connected
!
end
```
### Проверка работы протокола BGP

1. Проверим, поднялись ли соедские отношения на примере Spine_0
````
Spine_0#show bgp summary
BGP summary information for VRF default
Router identifier 10.2.0.0, local AS number 64512
Neighbor          AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
-------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.2.2.0       64512 Established   IPv4 Unicast            Negotiated              1          1
10.2.2.2       64512 Established   IPv4 Unicast            Negotiated              1          1
10.2.2.4       64512 Established   IPv4 Unicast            Negotiated              1          1
10.3.1.0       64512 Established   IPv4 Unicast            Negotiated              1          1
10.3.1.0       64512 Established   L2VPN EVPN              Negotiated              2          2
10.3.5.0       64512 Established   IPv4 Unicast            Negotiated              1          1
10.3.5.0       64512 Established   L2VPN EVPN              Negotiated              1          1
10.3.9.0       64512 Established   IPv4 Unicast            Negotiated              1          1
10.3.9.0       64512 Established   L2VPN EVPN              Negotiated              3          3


        
````
Как видим, соседство в состоянии Established для всех 3х Leaf. Более того, мы видит по 3 сессии для каждого Leaf. По одной сессия с AFI/SAFI IPV4 для underlay и overlay. И одна сессия для передачи L2VPN AFI/SAFI

2. Взглянем на таблицу маршрутизации Spine_0 для EVPN
````
Spine_0#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.2.0.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.3.0.0:10 mac-ip 0050.7966.6806
                                 10.3.1.0              -       100     0       i
 * >      RD: 10.3.0.0:10 mac-ip 0050.7966.6806 192.168.1.1
                                 10.3.1.0              -       100     0       i
 * >      RD: 10.3.4.0:20 mac-ip 0050.7966.6807
                                 10.3.5.0              -       100     0       i
 * >      RD: 10.3.4.0:20 mac-ip 0050.7966.6807 192.168.2.2
                                 10.3.5.0              -       100     0       i
 * >      RD: 10.3.8.0:10 mac-ip 0050.7966.6808
                                 10.3.9.0              -       100     0       i
 * >      RD: 10.3.8.0:10 mac-ip 0050.7966.6808 192.168.1.3
                                 10.3.9.0              -       100     0       i
 * >      RD: 10.3.8.0:30 mac-ip 0050.7966.6809
                                 10.3.9.0              -       100     0       i
 * >      RD: 10.3.8.0:30 mac-ip 0050.7966.6809 192.168.3.4
                                 10.3.9.0              -       100     0       i

Spine_0#show bgp evpn route-type imet 
BGP routing table information for VRF default
Router identifier 10.2.0.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.3.0.0:10 imet 10.3.1.0
                                 10.3.1.0              -       100     0       i
 * >      RD: 10.3.4.0:20 imet 10.3.5.0
                                 10.3.5.0              -       100     0       i
 * >      RD: 10.3.8.0:10 imet 10.3.9.0
                                 10.3.9.0              -       100     0       i
 * >      RD: 10.3.8.0:20 imet 10.3.9.0
                                 10.3.9.0              -       100     0       i
Spine_0#


 `````
 Мы видим два типа маршрутов Type-3 (imet), чтобы знать куда отправлять BUM трафик, например ARP. А так же тип маршрутов Type-2 (mac-ip). Так как в данном случае мы передаем MAC адреса, то видим мак-адреса наших клиентов. RD позволяет их отличать друг от друга. Кроме того, в таблице появились маршруты типа mac - mac. Данные маршруты передают IP адреса конечных устройств и их маки c целью дальнейшей маршрутизации

3. Что же у нас на уровне Leaf на примере Leaf_2?

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
