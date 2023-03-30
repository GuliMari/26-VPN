# 26-VPN:
1. Между двумя виртуалками поднять vpn в режимах:
- tun
- tap
Описать в чём разница, замерить скорость между виртуальными машинами в туннелях, сделать вывод об отличающихся показателях скорости.
2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на виртуалку.
3 (*). Самостоятельно изучить, поднять ocserv и подключиться с хоста к виртуалке.



## 1. Между двумя виртуалками поднять vpn в режимах: tun и  tap

TUN и TAP представляют собой программные сетевые устройства. Разница заключается в том, что TAP эмулирует Ethernet устройство и работает на канальном уровне модели OSI (L2), оперируя кадрами Ethernet, а TUN (сетевой туннель) работает на сетевом уровне модели OSI (L3), оперируя IP пакетами. TAP используется для создания сетевого моста, тогда как TUN для маршрутизации. 

Протестируем скорость в режиме `tap`:
```bash
[vagrant@server ~]$ iperf3 -s &
[1] 24964
[vagrant@server ~]$ -----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.10.2, port 38696
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 38698
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  13.2 MBytes   111 Mbits/sec                  
[  5]   1.00-2.00   sec  14.2 MBytes   119 Mbits/sec                  
[  5]   2.00-3.00   sec  15.2 MBytes   128 Mbits/sec                  
...                
[  5]  39.00-40.00  sec  16.0 MBytes   134 Mbits/sec                  
[  5]  40.00-40.02  sec   307 KBytes   105 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-40.02  sec   603 MBytes   126 Mbits/sec                  receiver
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
```

```bash
[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 38698 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  73.4 MBytes   123 Mbits/sec   36   82.6 KBytes       
[  5]   5.00-10.01  sec  76.0 MBytes   127 Mbits/sec   32    117 KBytes       
[  5]  10.01-15.00  sec  74.6 MBytes   125 Mbits/sec   28    104 KBytes       
[  5]  15.00-20.00  sec  75.5 MBytes   127 Mbits/sec   28   87.7 KBytes       
[  5]  20.00-25.00  sec  78.4 MBytes   132 Mbits/sec   36    103 KBytes       
[  5]  25.00-30.00  sec  73.5 MBytes   123 Mbits/sec   29   95.5 KBytes       
[  5]  30.00-35.00  sec  78.0 MBytes   131 Mbits/sec   21    112 KBytes       
[  5]  35.00-40.01  sec  74.2 MBytes   124 Mbits/sec   43   82.6 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.01  sec   604 MBytes   127 Mbits/sec  253             sender
[  5]   0.00-40.02  sec   603 MBytes   126 Mbits/sec                  receiver

iperf Done.
```
Скорость в режиме `tun`:
```bash
[root@server ~]# iperf3 -s &
[1] 24985
[root@server ~]# -----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
[root@server ~]# Accepted connection from 10.10.10.2, port 57358
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 57360
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  14.1 MBytes   118 Mbits/sec                  
[  5]   1.00-2.00   sec  14.6 MBytes   122 Mbits/sec                  
[  5]   2.00-3.00   sec  15.1 MBytes   127 Mbits/sec                  
[  5]   3.00-4.00   sec  15.0 MBytes   126 Mbits/sec                  
...              
[  5]  38.00-39.00  sec  15.3 MBytes   129 Mbits/sec                  
[  5]  39.00-40.00  sec  15.4 MBytes   129 Mbits/sec                  
[  5]  40.00-40.01  sec  39.6 KBytes  31.3 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-40.01  sec   609 MBytes   128 Mbits/sec                  receiver
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
```
```bash
[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 57360 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.01   sec  74.3 MBytes   124 Mbits/sec   29    120 KBytes       
[  5]   5.01-10.01  sec  76.5 MBytes   128 Mbits/sec   25    107 KBytes       
[  5]  10.01-15.01  sec  75.5 MBytes   127 Mbits/sec   26   85.9 KBytes       
[  5]  15.01-20.01  sec  77.5 MBytes   130 Mbits/sec   34   85.9 KBytes       
[  5]  20.01-25.00  sec  78.2 MBytes   131 Mbits/sec   25   89.8 KBytes       
[  5]  25.00-30.01  sec  76.3 MBytes   128 Mbits/sec   36   84.6 KBytes       
[  5]  30.01-35.01  sec  74.6 MBytes   125 Mbits/sec   28    106 KBytes       
[  5]  35.01-40.00  sec  76.6 MBytes   129 Mbits/sec   32   89.8 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec   610 MBytes   128 Mbits/sec  235             sender
[  5]   0.00-40.01  sec   609 MBytes   128 Mbits/sec                  receiver

iperf Done.
```
Разница в скорости в данном примере незначительна, но режим `tun` производительней.

## 2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на виртуалку.

