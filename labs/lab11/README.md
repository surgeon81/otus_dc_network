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
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip routing
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
```

 #### [Настройка Leaf_1](Leaf_1.cfg)

```
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
!
ip routing
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
```

 #### [Настройка Leaf_2](Leaf_2.cfg)

```
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
!
ip routing
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
   vlan 20
      rd 10.3.8.0:20
      route-target both 64512:10020
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.9.0/24
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
 * >      RD: 10.3.4.0:20 mac-ip 0050.7966.6807
                                 10.3.5.0              -       100     0       i
 * >      RD: 10.3.8.0:10 mac-ip 0050.7966.6808
                                 10.3.9.0              -       100     0       i
 * >      RD: 10.3.8.0:20 mac-ip 0050.7966.6809
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
 Мы видим два типа маршрутов Type-3 (imet), чтобы знать куда отправлять BUM трафик, например ARP. А так же тип маршрутов Type-2 (mac-ip). Так как в данном случае мы передаем MAC адреса, то видим мак-адреса наших клиентов. RD позволяет их отличать друг от друга

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

4. Маршруты на уровне Leaf.
````
Leaf_2# show bgp evpn route-type mac-ip 
BGP routing table information for VRF default
Router identifier 10.3.8.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.3.0.0:10 mac-ip 0050.7966.6806
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.0.0 
 *  ec    RD: 10.3.0.0:10 mac-ip 0050.7966.6806
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.4.0 
 * >Ec    RD: 10.3.4.0:20 mac-ip 0050.7966.6807
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 
 *  ec    RD: 10.3.4.0:20 mac-ip 0050.7966.6807
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.4.0 
 * >      RD: 10.3.8.0:10 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 10.3.8.0:20 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
Leaf_2# show bgp evpn route-type imet 
BGP routing table information for VRF default
Router identifier 10.3.8.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.3.0.0:10 imet 10.3.1.0
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.0.0 
 *  ec    RD: 10.3.0.0:10 imet 10.3.1.0
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.4.0 
 * >Ec    RD: 10.3.4.0:20 imet 10.3.5.0
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 
 *  ec    RD: 10.3.4.0:20 imet 10.3.5.0
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.4.0 
 * >      RD: 10.3.8.0:10 imet 10.3.9.0
                                 -                     -       -       0       i
 * >      RD: 10.3.8.0:20 imet 10.3.9.0
                                 -                     -       -       0       i
Leaf_2#
````
Тут уже видим, что имеется два равнозначных маршрута для каждого полученного мак-адреса. Что-же у нас с мак-таблицей


````
Leaf_2#show mac address-table 
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.6806    DYNAMIC     Vx1        1       0:41:58 ago
  10    0050.7966.6808    DYNAMIC     Et1        1       0:41:22 ago
  20    0050.7966.6807    DYNAMIC     Vx1        1       0:21:50 ago
  20    0050.7966.6809    DYNAMIC     Et2        1       0:21:50 ago
Total Mac Addresses for this criterion: 4
````
Как видим мы получам мак-адреса удаленных клиентов через интерфейс VxLAN 1 и локально через Eth порты

6. Проверим состояние VxLAN итерфейса
````
Leaf_2#show interfaces vxlan 1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback100 and is active with 10.3.9.0
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is 
    [10, 10010]       [20, 10020]      
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is not configured
  Headend replication flood vtep list is:
    10 10.3.1.0       
    20 10.3.5.0       
  Shared Router MAC is 0000.0000.0000
````
Как мы видим по параметру "Headend replication flood vtep list is" VxLAN туннель у нас установлен с Leaf_1 для VLAN 10 и Leaf_2 для VLAN 20

7. Ну, и проверим связность клиентов на примере Client_1
````
VPCS> ping 192.168.1.3        

84 bytes from 192.168.1.3 icmp_seq=1 ttl=64 time=49.756 ms
84 bytes from 192.168.1.3 icmp_seq=2 ttl=64 time=33.073 ms
84 bytes from 192.168.1.3 icmp_seq=3 ttl=64 time=41.629 ms
84 bytes from 192.168.1.3 icmp_seq=4 ttl=64 time=34.794 ms
84 bytes from 192.168.1.3 icmp_seq=5 ttl=64 time=37.916 ms

VPCS> 
````
