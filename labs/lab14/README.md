# VxLAN. EVPN L2. Multihoming

### Цели
- Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming 


### Топология сети
![VxLAN-MLAG](/labs/lab14/image.png)

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
interface Loopback0
   ip address 10.2.0.0/32
!
ip route 10.2.0.0/22 Null0
!
router bgp 64512
   router-id 10.2.0.0
   maximum-paths 4 ecmp 4
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback0
   neighbor evpn route-reflector-client
   neighbor evpn password 7 87NCY2eLqQQ=
   neighbor evpn send-community extended
   neighbor underlay peer group
   neighbor underlay remote-as 64512
   neighbor underlay route-reflector-client
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.2.0 peer group underlay
   neighbor 10.2.2.0 description Under_Leaf-0_0
   neighbor 10.2.2.2 peer group underlay
   neighbor 10.2.2.2 description Under_Leaf-0_1
   neighbor 10.2.2.4 peer group underlay
   neighbor 10.2.2.4 description Under_Leaf-1_0
   neighbor 10.2.2.6 peer group underlay
   neighbor 10.2.2.6 description Under_Leaf-1_1
   neighbor 10.3.0.0 peer group evpn
   neighbor 10.3.0.0 description Over_Leaf-0_0
   neighbor 10.3.0.1 peer group evpn
   neighbor 10.3.0.1 description Over_Leaf-0_1
   neighbor 10.3.4.0 peer group evpn
   neighbor 10.3.4.0 description Over_Leaf-1_0
   neighbor 10.3.4.1 peer group evpn
   neighbor 10.3.4.1 description Over_Leaf-1_1
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.2.0.0/22
!



```   
#### [Настройка Spine_1](Spine_1.cfg)

```

interface Loopback0
   ip address 10.2.4.0/32
!
ip routing
!
ip route 10.2.4.0/22 Null0
!
router bgp 64512
   router-id 10.2.4.0
   maximum-paths 4 ecmp 64
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback0
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
   neighbor 10.2.6.6 peer group underlay
   neighbor 10.2.6.6 description Leaf_3
   neighbor 10.3.0.0 peer group evpn
   neighbor 10.3.0.0 description Over_Leaf-0_0
   neighbor 10.3.0.1 peer group evpn
   neighbor 10.3.0.1 description Over_Leaf-0_1
   neighbor 10.3.4.0 peer group evpn
   neighbor 10.3.4.0 description Over_Leaf-1_0
   neighbor 10.3.4.1 peer group evpn
   neighbor 10.3.4.1 description Over_Leaf-1_1
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.2.4.0/22
!
end

```

#### [Настройка Leaf-0_0](Leaf-0_0.cfg)

```
!
link tracking group EVPN-MLAG-MH
   recovery delay 60
!
vlan 10
   name App1
!
vlan 4093-4094
   trunk group mlag-peer
!
vrf instance mgmt
!
interface Port-Channel1
   switchport access vlan 10
   mlag 10
   link tracking group EVPN-MLAG-MH downstream
!
interface Port-Channel3
   description Leaf-0_1:Po3
   mtu 9214
   switchport mode trunk
   switchport trunk group mlag-peer
!
interface Ethernet1
   description SW_1:Eth7
   switchport access vlan 10
   channel-group 1 mode active
!
interface Ethernet3
   description Leaf-1_0:Eth3
   mtu 9214
   channel-group 3 mode active
!
interface Ethernet4
   description Leaf-0_1:Eth4
   mtu 9214
   channel-group 3 mode active
!
interface Ethernet9
   description Spine_0:Eth1
   mtu 9214
   no switchport
   ip address 10.2.2.0/31
   link tracking group EVPN-MLAG-MH upstream
!
interface Ethernet10
   description Spine_1:Eth1
   mtu 9214
   no switchport
   ip address 10.2.6.0/31
   link tracking group EVPN-MLAG-MH upstream
!
interface Loopback0
   ip address 10.3.0.0/32
!
interface Loopback100
   ip address 10.3.1.0/32
!
interface Management0
   vrf mgmt
   ip address 172.16.0.1/24
!
interface Vlan4093
   ip address 10.3.0.128/31
!
interface Vlan4094
   no autostate
   ip address 192.168.0.0/31
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip virtual-router mac-address 00:00:80:00:00:00
!
ip routing
no ip routing vrf mgmt
!
mlag configuration
   domain-id Rack-1
   local-interface Vlan4094
   peer-address 192.168.0.1
   peer-address heartbeat 172.16.0.2 vrf mgmt
   peer-link Port-Channel3
   dual-primary detection delay 10 action errdisable all-interfaces
!
router bgp 64512
   router-id 10.3.0.0
   maximum-paths 4 ecmp 64
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback0
   neighbor evpn password 7 87NCY2eLqQQ=
   neighbor evpn send-community standard extended
   neighbor over peer group
   neighbor overlay peer group
   neighbor underlay peer group
   neighbor underlay remote-as 64512
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.0.0 peer group evpn
   neighbor 10.2.0.0 description Over_Spine_0
   neighbor 10.2.2.1 peer group underlay
   neighbor 10.2.2.1 description Under_Spine_0
   neighbor 10.2.4.0 peer group evpn
   neighbor 10.2.4.0 description Over_Spine_1
   neighbor 10.2.6.1 peer group underlay
   neighbor 10.2.6.1 description Under_Spine_0
   neighbor 10.3.0.129 peer group underlay
   neighbor 10.3.0.129 description Under_Leaf-0_1
   !
   vlan 10
      rd 10.3.1.0:10
      route-target both 64512:10010
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.0.0/32
      network 10.3.0.128/31
      network 10.3.1.0/32
!
end

```

 #### [Настройка Leaf-0_1](Leaf-0_1.cfg)

```
!
link tracking group EVPN-MLAG-MH
   recovery delay 60
!
vlan 10
   name App1
!
vlan 4093-4094
   trunk group mlag-peer
!
vrf instance mgmt
!
interface Port-Channel1
   switchport access vlan 10
   mlag 10
   link tracking group EVPN-MLAG-MH downstream
!
interface Port-Channel3
   description Leaf-0_0:Po3
   mtu 9214
   switchport mode trunk
   switchport trunk group mlag-peer
!
interface Ethernet1
   description SW_1:Eth8
   switchport access vlan 10
   channel-group 1 mode active
!
interface Ethernet3
   description Leaf-0_0:Eth3
   mtu 9214
   channel-group 3 mode active
!
interface Ethernet4
   description Leaf-0_0:Eth4
   mtu 9214
   channel-group 3 mode active
!
interface Ethernet9
   description Spine_0:Eth2
   mtu 9214
   no switchport
   ip address 10.2.2.2/31
   link tracking group EVPN-MLAG-MH upstream
!
interface Ethernet10
   description Spine_1:Eth2
   mtu 9214
   no switchport
   ip address 10.2.6.2/31
   link tracking group EVPN-MLAG-MH upstream
!
interface Loopback0
   ip address 10.3.0.1/32
!
interface Loopback100
   ip address 10.3.1.0/32
!
interface Management0
   vrf mgmt
   ip address 172.16.0.2/24
!
interface Vlan4093
   ip address 10.3.0.129/31
!
interface Vlan4094
   no autostate
   ip address 192.168.0.1/31
!
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip virtual-router mac-address 00:00:80:00:00:00
!
ip routing
no ip routing vrf mgmt
!
mlag configuration
   domain-id Rack-1
   local-interface Vlan4094
   peer-address 192.168.0.0
   peer-address heartbeat 172.16.0.1 vrf mgmt
   peer-link Port-Channel3
   dual-primary detection delay 10 action errdisable all-interfaces
!
router bgp 64512
   router-id 10.3.0.1
   maximum-paths 4 ecmp 64
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback0
   neighbor evpn password 7 87NCY2eLqQQ=
   neighbor evpn send-community standard extended
   neighbor underlay peer group
   neighbor underlay remote-as 64512
   neighbor underlay password 7 ftDxwZDN4HE=
   neighbor 10.2.0.0 peer group evpn
   neighbor 10.2.0.0 description Over_Spine_0
   neighbor 10.2.2.3 peer group underlay
   neighbor 10.2.2.3 description Under_Spine_0
   neighbor 10.2.4.0 peer group evpn
   neighbor 10.2.4.0 description Over_Spine_1
   neighbor 10.2.6.3 peer group underlay
   neighbor 10.2.6.3 description Under_Spine_0
   neighbor 10.3.0.128 peer group underlay
   neighbor 10.3.0.128 description Under_Leaf-0_1
   !
   vlan 10
      rd 10.3.1.0:10
      route-target both 64512:10010
      redistribute learned
   !
   address-family evpn
      neighbor evpn activate
   !
   address-family ipv4
      network 10.3.0.1/32
      network 10.3.0.128/31
      network 10.3.1.0/32
!
end

```

 Настройка [Leaf-1_0](Leaf-1_0.cfg) и [Leaf-1_1](Leaf-1_1.cfg) аналогична двум предыдущим Leaf

### Проверка работы MLAG
1. Проверим, видят ли друг друга Leaf ли соедские отношения на примере Leaf-0_0
````
Leaf-0_0#show mlag
MLAG Configuration:              
domain-id                          :              Rack-1
local-interface                    :            Vlan4094
peer-address                       :         192.168.0.1
peer-link                          :       Port-Channel3
hb-peer-address                    :          172.16.0.2
hb-peer-vrf                        :                mgmt
peer-config                        :          consistent
                                                       
MLAG Status:                     
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:03:37:66
dual-primary detection             :          Configured
dual-primary interface errdisabled :               False
                                                       
MLAG Ports:                      
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1
        
````
Как видим, состояние MLAG активно и пиры видят друг друга

2. Проверим MAC-таблицу на коммутатора

````
Leaf-0_0#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
   1    0000.8000.0000    STATIC      Po3
  10    0000.8000.0000    STATIC      Cpu
  10    0050.7966.6806    DYNAMIC     Po1        1       0:00:12 ago
  10    0050.7966.6809    DYNAMIC     Vx1        1       0:00:10 ago
4093    0000.8000.0000    STATIC      Cpu
4094    0000.8000.0000    STATIC      Cpu
Total Mac Addresses for this criterion: 8


Leaf-0_1(config-if-Et9-10)#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
   1    0000.8000.0000    STATIC      Cpu
  10    0000.8000.0000    STATIC      Cpu
  10    0050.7966.6806    DYNAMIC     Po1        1       0:00:06 ago
  10    0050.7966.6809    DYNAMIC     Vx1        1       0:00:05 ago
4093    0000.8000.0000    STATIC      Cpu
4094    0000.8000.0000    STATIC      Cpu
Total Mac Addresses for this criterion: 6
````

Таблица на обоих MLAG соседах одинаковая

### Проверка работы BGP
1. Проверим состояние BGP на примере Leaf-0_0

````
Leaf-0_0#show bgp summary 
BGP summary information for VRF default
Router identifier 10.3.0.0, local AS number 64512
Neighbor            AS Session State AFI/SAFI                AFI/SAFI State   NLRI Rcd   NLRI Acc
---------- ----------- ------------- ----------------------- -------------- ---------- ----------
10.2.0.0         64512 Established   IPv4 Unicast            Negotiated              6          6
10.2.0.0         64512 Established   L2VPN EVPN              Negotiated              1          1
10.2.2.1         64512 Established   IPv4 Unicast            Negotiated              6          6
10.2.4.0         64512 Established   IPv4 Unicast            Negotiated              6          6
10.2.4.0         64512 Established   L2VPN EVPN              Negotiated              1          1
10.2.6.1         64512 Established   IPv4 Unicast            Negotiated              6          6
10.3.0.129       64512 Established   IPv4 Unicast            Negotiated              3          3

````


 Мы видим, что соседства со Spine свичами и MLAG соседом установлено как для IPv4 так и для L2VPN AFI/SAFI


2. Маршруты для L2VPN отличаются на обоих MLAG пирах. На втором пире, присутстуют mac-ip маршруты конечного хоста, полученные через MP-BGP

````
Leaf-0_0#show bgp evpn route-type imet 
BGP routing table information for VRF default
Router identifier 10.3.0.0, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.3.1.0:10 imet 10.3.1.0
                                 -                     -       -       0       i
 * >Ec    RD: 10.3.5.0:10 imet 10.3.5.0
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.1 C-LST: 10.2.4.0 
 *  ec    RD: 10.3.5.0:10 imet 10.3.5.0
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 



Leaf-0_1(config)#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.3.0.1, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.3.1.0:10 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
          RD: 10.3.1.0:10 mac-ip 0050.7966.6806
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.0.0 
          RD: 10.3.1.0:10 mac-ip 0050.7966.6806
                                 10.3.1.0              -       100     0       i Or-ID: 10.3.0.0 C-LST: 10.2.4.0 
 * >Ec    RD: 10.3.5.0:10 mac-ip 0050.7966.6809
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.4.0 
 *  ec    RD: 10.3.5.0:10 mac-ip 0050.7966.6809
                                 10.3.5.0              -       100     0       i Or-ID: 10.3.4.0 C-LST: 10.2.0.0 
````

3. Ну, и проверим связность клиентов на примере Client_1 одновременно отключая различные линки

````
Client_1> ping 192.168.1.4 -c 9999

84 bytes from 192.168.1.4 icmp_seq=1 ttl=64 time=164.316 ms
84 bytes from 192.168.1.4 icmp_seq=2 ttl=64 time=260.320 ms
192.168.1.4 icmp_seq=3 timeout
192.168.1.4 icmp_seq=4 timeout
192.168.1.4 icmp_seq=5 timeout
84 bytes from 192.168.1.4 icmp_seq=6 ttl=64 time=303.208 ms
84 bytes from 192.168.1.4 icmp_seq=7 ttl=64 time=146.779 ms
84 bytes from 192.168.1.4 icmp_seq=8 ttl=64 time=369.266 ms
84 bytes from 192.168.1.4 icmp_seq=9 ttl=64 time=182.319 ms
84 bytes from 192.168.1.4 icmp_seq=10 ttl=64 time=145.042 ms
84 bytes from 192.168.1.4 icmp_seq=11 ttl=64 time=314.286 ms
84 bytes from 192.168.1.4 icmp_seq=12 ttl=64 time=419.650 ms
84 bytes from 192.168.1.4 icmp_seq=13 ttl=64 time=301.741 ms
84 bytes from 192.168.1.4 icmp_seq=14 ttl=64 time=206.088 ms
84 bytes from 192.168.1.4 icmp_seq=15 ttl=64 time=947.955 ms
84 bytes from 192.168.1.4 icmp_seq=16 ttl=64 time=217.366 ms
84 bytes from 192.168.1.4 icmp_seq=17 ttl=64 time=406.866 ms
84 bytes from 192.168.1.4 icmp_seq=18 ttl=64 time=176.750 ms
84 bytes from 192.168.1.4 icmp_seq=19 ttl=64 time=140.207 ms
84 bytes from 192.168.1.4 icmp_seq=20 ttl=64 time=391.621 ms
84 bytes from 192.168.1.4 icmp_seq=21 ttl=64 time=341.675 ms
84 bytes from 192.168.1.4 icmp_seq=22 ttl=64 time=505.117 ms
192.168.1.4 icmp_seq=23 timeout
84 bytes from 192.168.1.4 icmp_seq=24 ttl=64 time=246.518 ms
84 bytes from 192.168.1.4 icmp_seq=25 ttl=64 time=190.895 ms
84 bytes from 192.168.1.4 icmp_seq=26 ttl=64 time=212.601 ms
84 bytes from 192.168.1.4 icmp_seq=27 ttl=64 time=270.879 ms
VPCS> 
````
как видим, связь нарушается не надолго, и трафик продолжает идти
