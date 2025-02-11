# Проектная работа "Миграция существующей инфраструктуры серверов на новую топологию CLOS"

### Описание проекта
Целью данной работы является отработка полученных навыков в проектировании и развертывании сети ЦОД c использованием VxLAN.
Данная работа рассматривает реальную задачу переноса существующей инфраструктуры сервисов в новый ЦОД.
К новой сети предъявляются следующие требования:
1. Обеспечение резервирования подключения серверов
2. Обеспечить возможность L3 коммуникации в пределах фабрики
3. Изолировать между собой трафики сети управления, DMZ и внутренних сервисов.
4. Изолировать L2/L3 взаимодействие между стойками. Исключение - VLAN управления.
5. Использовать возможности Active-Active для выхода трафика из фабрики через кластер фаерволов
6. Для предовращения нарушения сервисов при падении всех линков от Leaf до Spine коммутаторов использована технология Arista "link tracking group". Данный функционал позволяет переводить необходимые интерфейсы в "errdisable" состояние при падении зависимых портов. Таким образом проблемный коммутатор полностью выводится из MLAG группы


### Ограничения среды
- Выключен BFD в связи с низкой производительностью виртуальной среды
- Для уменьшения конфигурационных файлов выключена аутентификация протоколов маршрутизации
- В качестве нодов кластера а так же фаерволов использовался образ коммутарора Arista
- Для симуляции Active-Active кластера фаерволов использовался т.н VRRP passive
- В связи с ограниченным бюджетом, существующая топология пары коммутаторов Arista с подключенными фаерволам и существующими испльзуются в качестве Spine а так-же Border Spine

### Выделение адресного пространства
С целью более плавного перехода а также минимизации прерывания сервисов, в IP план была заложена существующая система разделения на VLAN и подсети существущих сервисов.
- Префикс 172.16.x.0/24 где х, равен номеру VLAN.
- VLAN 10 - сеть управления.
- VLAN 20 - сеть для внутренних ресурсов.
- VLAN 30 - DMZ
- VRRP IP 172.16.x.254

На оборудовании используется два Loopback адреса
- Loopback0 - используется в качестве Router ID, части  так же для организации BGP сессии overlay сети
- Loopack100 - одинаковый для MLAG пары. Служит в качестве интерфейса для терминации VxLAN туннелей а так же в части Route Distinguisher 

Все линки между коммутаторами используют /31 подсети.
Для Underlay сети используется OSPF.
Для Overlay сети - iBGP

Для каждого Spine/Leaf зарезервирован префикс /22
- Первый диапазон /24 предназначен для Loopack underlay. Последний /31 диапазаон для OSPF стыка между двумя MLAG пирами
- Второй диапазон /24 - для Loopback overlay
- Третий диапазон у Spine - для p2p связи с Leaf (нечетный на стороне Spine). У Leaf для клиентских сервисов
- Четверный диапазон заререзвирован у всех


### Выделение VNI, RD, и RT, OSPF area
- VNI: 1xxxx - где xxxx номер VLAN
- RD: Используется type 1 и соостоит из Loopback0 IP адреса и номера VNI
- RT: соостоит из номер AS и измененного номера VNI yxxxx (y - номер стойки, xxxx - номер VLAN)
- Все Spine находятся в области 0.0.0.0
- Все Leaf находятся в области равным Loopack 100

### Адреса
#### Адреса коммутатора Spine 0
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.0.0|32|Loopback 0|RID,RD
10.2.1.0|32|Loopback 100|VxLAN endpoint
10.2.2.1|31|Eth 1|p2p leaf 00
10.2.2.3|31|Eth 2|p2p leaf 01
10.2.2.5|31|Eth 3|p2p leaf 10
10.2.2.7|31|Eth 4|p2p leaf 11

#### Адреса коммутатора Spine 1
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.2.4.0|32|Loopback 0|RID,RD
10.2.5.0|32|Loopback 100|VxLAN endpoint
10.2.6.1|31|Eth 1|p2p leaf 00
10.2.6.3|31|Eth 2|p2p leaf 01
10.2.6.5|31|Eth 3|p2p leaf 10
10.2.6.7|31|Eth 4|p2p leaf 11


#### Адреса коммутатора Leaf 00
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.0.0|32|Loopback 0|RID,RD
10.3.1.0|32|Loopback 100|VxLAN endpoint
10.2.2.0|31|Eth 9|p2p spine 0
10.2.6.0|31|Eth 10|p2p spine 1
10.3.0.254|31|VLAN 4093|MLAG peering OSPF
192.168.0.0|31|VLAN 4094|MLAG peering

#### Адреса коммутатора Leaf 01
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.0.1|32|Loopback 0|RID,RD
10.3.1.0|32|Loopback 100|VxLAN endpoint
10.2.2.2|31|Eth 9|p2p spine 0
10.2.6.2|31|Eth 10|p2p spine 1
10.3.0.255|31|VLAN 4093|MLAG peering OSPF
192.168.0.1|31|VLAN 4094|MLAG peering


#### Адреса коммутатора Leaf 10
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.4.0|32|Loopback 0|RID,RD
10.3.5.0|32|Loopback 100|VxLAN endpoint
10.2.2.4|31|Eth 9|p2p spine 0
10.2.6.4|31|Eth 10|p2p spine 1
10.3.4.254|31|VLAN 4093|MLAG peering OSPF
192.168.0.0|31|VLAN 4094|MLAG peering

#### Адреса коммутатора Leaf 11
IP|маска|Интерфейс|Назначение
|---|---|---|---|
10.3.4.1|32|Loopback 0|RID,RD
10.3.5.0|32|Loopback 100|VxLAN endpoint
10.2.2.6|31|Eth 9|p2p spine 0
10.2.6.6|31|Eth 10|p2p spine 1
10.3.4.255|31|VLAN 4093|MLAG peering OSPF
192.168.0.1|31|VLAN 4094|MLAG peering


### Топология сети
##### Планируема топология
![Планируемая топология](/labs/Курсовая_работа/network.png)

##### Топология на стенде
![Топология стенда](/labs/Курсовая_работа/lab.png)



### Подготовка стенда
[Конфигурация Spine_0](/labs/Курсовая_работа/Spine_0.cfg)

[Конфигурация Spine_1](/labs/Курсовая_работа/Spine_1.cfg)

[Конфигурация Leaf_00](/labs/Курсовая_работа/Leaf_00.cfg)

[Конфигурация Leaf_01](/labs/Курсовая_работа/Leaf_01.cfg)

[Конфигурация Leaf_10](/labs/Курсовая_работа/Leaf_10.cfg)

[Конфигурация Leaf_11](/labs/Курсовая_работа/Leaf_11.cfg)


#### Конфигурация Underaly уровня Spine

```
router ospf 1
   router-id <Loopback0>
   passive-interface default
   no passive-interface <ports_to_leaf>
!
interface Loopback0
   ip ospf area 0.0.0.0
!
interface Loopback100
   ip ospf area 0.0.0.0
!
interface <port_to_leaf>
   ip ospf area 0.0.0.0
!

```
#### Конфигурация Underaly уровня Leaf
```
router ospf 1
   router-id <Loopback0>
   passive-interface default
   no passive-interface <ports_to_spines>
   area <Loopback100> nssa no-summary
   area <Loopback100> range <area subnet>
   max-lsa 12000
!
interface Loopback0
   ip ospf area <Loopback100>
!
interface Loopback100
   ip ospf area <Loopback100>
!
interface <port_to_spine>
   ip ospf area 0.0.0.0
!
interface 4093
   ip ospf area <Loopback100>

```
#### Конфигурация Overlay уровня Spine
```
router bgp <ASN>
   router-id <Loopback0>
   maximum-paths 4 ecmp 4
   neighbor evpn peer group
   neighbor evpn remote-as 64512
   neighbor evpn update-source Loopback0
   neighbor evpn route-reflector-client
   neighbor evpn send-community extended
   neighbor <Leaf1 Loopback0> peer group evpn
   ...
   neighbor <LeafX Loopback0> peer group evpn
   !
   address-family evpn
      neighbor evpn activate
```
#### Конфигурация Overlay уровня Leaf
```
router bgp <ASN>
   router-id <Loobpack0>
   maximum-paths 4 ecmp 4
   neighbor evpn peer group
   neighbor evpn remote-as <ASN>
   neighbor evpn update-source Loopback0
   neighbor evpn send-community standard extended
   neighbor <Spine1 Loopback0> peer group evpn
   neighbor <Spine2 Loopback0> peer group evpn
   !
   address-family evpn
      neighbor evpn activate
```
#### Конфигурация VxLAN уровня Spine
```
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan <VLAN ID> vni <VNI>
   !
router bgp <ASN>
   vlan <VLAN ID>
      rd <Loopback100>:<VNI>
      route-target both <ASN>:<Rack1-VNI>
      route-target both <ASN>:<Rack2-VNI>
      redistribute learned
   !

```
#### Конфигурация VxLAN уровня Leaf
```
interface Vxlan1
   vxlan source-interface Loopback100
   vxlan udp-port 4789
   vxlan vlan <VLAN ID> vni <VNI>
   !
router bgp <ASN>
   vlan <VLAN ID>
      rd <Loopback100>:<VNI>
      route-target both <ASN>:<Rack-VNI>
      redistribute learned
   !
```
#### Конфигурация MLAG
```
mlag configuration
   domain-id Rack-<ID>
   local-interface Vlan4094
   peer-address 192.168.0.x
   peer-link Port-Channel3
!
interface Port-Channel3
   mtu 9214
   switchport mode trunk
   switchport trunk group mlag-peer
```

