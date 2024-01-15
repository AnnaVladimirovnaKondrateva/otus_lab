#  iBGP
### Топология
![](Схема1.png)

###  Цели

  1. Настроить iBGP в офисе Москва.
  2. Настроить iBGP в сети Триада.
  3. Организовать полную IP связность всех сетей  
  
  Описание задания:
  - Настроить iBGP в офисе Москва.
  - Настроить iBGP в провайдере Триада c использованием RR.
  - Настроить офис Москва так, чтобы приоритетным провайдером стал Ламас.
  - Настроить офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
  - Все сети должны иметь IP связность.
  
#### Часть 1. Подготовительная.
Данные для каждой локации, необходимые для настройки eBGP
|Локация| Номер AS     | Блок адресов    | 
|:-----------------|:---------------|-------------------------:|
| Москва  | 1001 | 100.1.0.0/16    |
| С.-Петербург  | 2042 | 100.2.0.0/16    |
| Киторн | 101 | 100.6.0.0/16    |
| Ламас | 301 | 100.7.0.0/16    |
| Триада | 520 | 100.3.0.0/16    | 
##### Укрупненный участок общей схемы с отображением некоторых настроек
![](Схема2.png)

 Таблица адресации

|Локация| Устройство     | Интерфейс    | IP адрес             | Маска подсети| 
|:-----------------|:---------------|-------------------------:|:--------------------|-------:|
| С.-Петербург  | R18| loopback 1   | 100.2.0.18 |255.255.255.255| |
| С.-Петербург  | R18| e0/2 |100.3.0.6 |255.255.255.252|
| С.-Петербург  | R18| e0/3 |10.3.0.10 |255.255.255.252|
| Москва  | R14|  loopback 1 |100.1.0.14 |255.255.255.255|
| Москва  | R14|  e0/2 |100.6.0.2 |255.255.255.252|
| Москва  | R15|  loopback 1 |100.1.0.15 |255.255.255.255|
| Москва  | R15|  e0/2 |100.7.0.2 |255.255.255.252|
| Киторн  | R22|  loopback 1 |100.6.0.22 |255.255.255.255|
| Киторн  | R22|  e0/0 |100.6.0.1 |255.255.255.252|
| Киторн  | R22|  e0/1 |100.7.0.6 |255.255.255.252|
| Ламас  | R21|  loopback 1 |100.7.0.21 |255.255.255.255|
| Ламас  | R21|  e0/0 |100.7.0.1 |255.255.255.252|
| Ламас  | R21|  e0/1 |100.7.0.5 |255.255.255.252|
| Ламас  | R21|  e0/2 |100.3.0.2 |255.255.255.252|
| Триада  | R24|  loopback 1 |100.3.0.24 |255.255.255.255|
| Триада  | R24|  e0/0 |100.3.0.1 |255.255.255.252|
| Триада  | R24|  e0/3 |100.3.0.5 |255.255.255.252|
| Триада  | R26|  loopback 1 |100.3.0.26 |255.255.255.255|
| Триада  | R26|  e0/3 |100.3.0.9 |255.255.255.252|

Интерфейсы роутеров настроила в соответствии с таблицей адресации. 


 
#### Часть 2. Настройка iBGP в офисе Москва между R14 R15

1. На R14 и R15 добавлю интерфейсы , смотрящие в сторону провайдера, в OSPF
```
R14#conf t
R14(config)#router ospf 1
R14(config-router)#no passive-interface e0/2
R14(config-router)#int e0/2
R14(config-if)#ip ospf 1 area 0
R14(config-if)#^Z
R14#
```
```
R15#
R15#conf t
R15(config)#router ospf 1
R15(config-router)#no pass e0/2
R15(config-router)#int e0/2
R15(config-if)#ip ospf 1 area 0
R15(config-if)#^Z
R15#
```
Убедилась, что ТМ R14, R15 содержит все нужные маршруты
```
R14#sh ip route ospf

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 20 subnets, 2 masks
O        10.1.0.12/32 [110/11] via 10.1.1.193, 3w6d, Ethernet0/0
O        10.1.0.13/32 [110/11] via 10.1.1.1, 3w6d, Ethernet0/1
O        10.1.0.15/32 [110/11] via 10.1.1.10, 3w6d, Ethernet1/0
O        10.1.0.19/32 [110/11] via 10.1.1.138, 3w6d, Ethernet0/3
O IA     10.1.0.20/32 [110/21] via 10.1.1.10, 3w6d, Ethernet1/0
O        10.1.1.12/30 [110/20] via 10.1.1.193, 4w4d, Ethernet0/0
                      [110/20] via 10.1.1.1, 4w4d, Ethernet0/1
O        10.1.1.16/30 [110/20] via 10.1.1.193, 4w0d, Ethernet0/0
                      [110/20] via 10.1.1.1, 4w0d, Ethernet0/1
O        10.1.1.20/30 [110/20] via 10.1.1.193, 4w0d, Ethernet0/0
                      [110/20] via 10.1.1.1, 4w0d, Ethernet0/1
O        10.1.1.128/30 [110/20] via 10.1.1.1, 4w4d, Ethernet0/1
O        10.1.1.224/30 [110/20] via 10.1.1.193, 4w4d, Ethernet0/0
O IA     10.1.1.248/30 [110/20] via 10.1.1.10, 4w4d, Ethernet1/0
      100.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
O        100.1.0.15/32 [110/11] via 10.1.1.10, 1w1d, Ethernet1/0
O        100.7.0.0/30 [110/20] via 10.1.1.10, 04:23:10, Ethernet1/0

```
```
R15#sh ip route ospf

Gateway of last resort is 10.1.1.9 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.1.1.9, 4w1d, Ethernet1/0
      10.0.0.0/8 is variably subnetted, 20 subnets, 2 masks
O        10.1.0.12/32 [110/11] via 10.1.1.225, 3w6d, Ethernet0/1
O        10.1.0.13/32 [110/11] via 10.1.1.129, 3w6d, Ethernet0/0
O        10.1.0.14/32 [110/11] via 10.1.1.9, 3w6d, Ethernet1/0
O IA     10.1.0.19/32 [110/21] via 10.1.1.9, 3w6d, Ethernet1/0
O        10.1.0.20/32 [110/11] via 10.1.1.249, 3w6d, Ethernet0/3
O        10.1.1.0/30 [110/20] via 10.1.1.129, 4w4d, Ethernet0/0
O        10.1.1.12/30 [110/20] via 10.1.1.225, 4w4d, Ethernet0/1
                      [110/20] via 10.1.1.129, 4w4d, Ethernet0/0
O        10.1.1.16/30 [110/20] via 10.1.1.225, 4w0d, Ethernet0/1
                      [110/20] via 10.1.1.129, 4w0d, Ethernet0/0
O        10.1.1.20/30 [110/20] via 10.1.1.225, 4w0d, Ethernet0/1
                      [110/20] via 10.1.1.129, 4w0d, Ethernet0/0
O IA     10.1.1.136/30 [110/20] via 10.1.1.9, 4w4d, Ethernet1/0
O        10.1.1.192/30 [110/20] via 10.1.1.225, 4w4d, Ethernet0/1
      100.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
O        100.1.0.14/32 [110/11] via 10.1.1.9, 1w1d, Ethernet1/0
O        100.6.0.0/30 [110/20] via 10.1.1.9, 04:25:23, Ethernet1/0
```
На этом этапе связность между Москвой и СПБ есть. Фиксирую, что R14 и R15 выходят в СПб через разных провайдеров (R14 через Киторн AS101) (R15 через Ламас AS301)
```
R14#ping 100.2.0.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 100.2.0.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R14#trace 100.2.0.18
Type escape sequence to abort.
Tracing the route to 100.2.0.18
VRF info: (vrf in name/id, vrf out name/id)
  1 100.6.0.1 [AS 101] 1 msec 0 msec 0 msec
  2 100.7.0.5 [AS 301] 1 msec 0 msec 0 msec
  3 100.3.0.1 [AS 520] 1 msec 0 msec 0 msec
  4 100.3.0.6 [AS 520] 1 msec *  1 msec
R14#

```
```
R15# ping 100.2.0.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 100.2.0.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#trace 100.2.0.18
Type escape sequence to abort.
Tracing the route to 100.2.0.18
VRF info: (vrf in name/id, vrf out name/id)
  1 100.7.0.1 [AS 301] 0 msec 0 msec 0 msec
  2 100.3.0.1 [AS 520] 0 msec 0 msec 1 msec
  3 100.3.0.6 [AS 520] 0 msec *  1 msec
R15#

```
2. На R14 и R15 настраиваю iBGP. для настройки iBGP использую Loopback 1 
```
R14#conf t
R14(config)#router bgp 1001
R14(config-router)#nei 100.1.0.15 remote 1001
R14(config-router)#nei 100.1.0.15 update loo 1
R14(config-router)#nei 100.1.0.15 next-hop-s
R14(config-router)#^Z
R14#
```
```
R15#conf t
R15(config)#router bgp 1001
R15(config-router)#nei 100.1.0.14 remote 1001
R15(config-router)#nei 100.1.0.14 update loo 1
R15(config-router)#nei 100.1.0.14 next-hop-s
R15(config-router)#^Z
R15#
```
Между R14 R15 установилось iBGP соседство
```
R14#sh ip bgp summ

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.1.0.15      4         1001      18      14       86    0    0 00:02:47        4
100.6.0.1       4          101   14270   14321       86    0    0 1w1d            4
```
R14 и R15 получили друг от друга маршруты и проинсталировали их в свои ТМ
```
R14#sh ip bgp

     Network          Next Hop            Metric LocPrf Weight Path
 * i 100.1.0.0/16     100.1.0.15               0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>i 100.2.0.0/16     100.1.0.15               0    100      0 301 520 2042 i
 *                    100.6.0.1                              0 101 301 520 2042 i
 *>i 100.3.0.0/16     100.1.0.15               0    100      0 301 520 i
 *                    100.6.0.1                              0 101 301 520 i
 *>  100.6.0.0/16     100.6.0.1                0             0 101 i
 *>i 100.7.0.0/16     100.1.0.15               0    100      0 301 i
 *                    100.6.0.1                              0 101 301 i
R14#
```
```
R14#sh ip route bgp

Gateway of last resort is 0.0.0.0 to network 0.0.0.0

      100.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
B        100.2.0.0/16 [200/0] via 100.1.0.15, 00:05:03
B        100.3.0.0/16 [200/0] via 100.1.0.15, 00:05:03
B        100.6.0.0/16 [20/0] via 100.6.0.1, 1w1d
B        100.7.0.0/16 [200/0] via 100.1.0.15, 00:05:03
```
```
R15#sh ip bgp

     Network          Next Hop            Metric LocPrf Weight Path
 * i 100.1.0.0/16     100.1.0.14               0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>  100.2.0.0/16     100.7.0.1                              0 301 520 2042 i
 *>  100.3.0.0/16     100.7.0.1                              0 301 520 i
 *>i 100.6.0.0/16     100.1.0.14               0    100      0 101 i
 *                    100.7.0.1                              0 301 101 i
 *>  100.7.0.0/16     100.7.0.1                0             0 301 i
R15#
```
```
R15#sh ip route bgp

Gateway of last resort is 10.1.1.9 to network 0.0.0.0

      100.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
B        100.2.0.0/16 [20/0] via 100.7.0.1, 1w1d
B        100.3.0.0/16 [20/0] via 100.7.0.1, 1w1d
B        100.6.0.0/16 [200/0] via 100.1.0.14, 00:09:11
B        100.7.0.0/16 [20/0] via 100.7.0.1, 1w1d
R15#
```
После этого связность R14-R18 оказалась нарушена. Пакет с R14 попадает на R15, но дальше не идет, хотя пинг R15-R18 успешен.
```
R14#ping 100.2.0.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 100.2.0.18, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R14#trace 100.2.0.18
Type escape sequence to abort.
Tracing the route to 100.2.0.18
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.1.10 1 msec 0 msec 0 msec
  2  *  *  *

```
ping R15-R18
```
R15#ping 100.2.0.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 100.2.0.18, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#trace 100.2.0.18
Type escape sequence to abort.
Tracing the route to 100.2.0.18
VRF info: (vrf in name/id, vrf out name/id)
  1 100.7.0.1 [AS 301] 0 msec 0 msec 0 msec
  2 100.3.0.1 [AS 520] 1 msec 0 msec 0 msec
  3 100.3.0.6 [AS 520] 1 msec *  0 msec
R15#

```
При этом в обратную сторону R18-R14 связность не нарушена.
```
R18#ping 100.1.0.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 100.1.0.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R18#trace 100.1.0.14
Type escape sequence to abort.
Tracing the route to 100.1.0.14
VRF info: (vrf in name/id, vrf out name/id)
  1 100.3.0.5 [AS 520] 0 msec 1 msec 0 msec
  2 100.3.0.2 [AS 520] 0 msec 1 msec 0 msec
  3 100.7.0.2 [AS 301] 0 msec 0 msec 1 msec
  4 10.1.1.9 0 msec 1 msec *
R18#

```

Configs can be found [here](configs/).
###  The End 