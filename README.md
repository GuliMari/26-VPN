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

Настраиваем RAS с помощью `ansible` и заходим на сервер:

```bash
[root@ras ~]# systemctl status openvpn@server.service 
● openvpn@server.service - OpenVPN Tunneling Application on server
   Loaded: loaded (/etc/systemd/system/openvpn@.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-03-31 14:10:03 MSK; 52s ago
 Main PID: 13878 (openvpn)
   Status: "Initialization Sequence Completed"
    Tasks: 1 (limit: 2749)
   Memory: 1.7M
   CGroup: /system.slice/system-openvpn.slice/openvpn@server.service
           └─13878 /usr/sbin/openvpn --cd /etc/openvpn/server --config server.conf

Mar 31 14:10:03 ras systemd[1]: Starting OpenVPN Tunneling Application on server...
Mar 31 14:10:03 ras systemd[1]: Started OpenVPN Tunneling Application on server.
[root@ras ~]#
[root@ras ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:03:15:fa brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
       valid_lft 86203sec preferred_lft 86203sec
    inet6 fe80::5054:ff:fe03:15fa/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:5a:de:13 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.10/24 brd 192.168.56.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe5a:de13/64 scope link 
       valid_lft forever preferred_lft forever
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.10.10.1 peer 10.10.10.2/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::d8b5:4435:1462:442b/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
[root@ras ~]#
[root@ras ~]# ip r
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
10.10.10.0/24 via 10.10.10.2 dev tun0 
10.10.10.2 dev tun0 proto kernel scope link src 10.10.10.1 
192.168.56.0/24 dev eth1 proto kernel scope link src 192.168.56.10 metric 101 
[root@ras ~]#
[root@ras ~]# tail -f /var/log/openvpn-status.log 
OpenVPN CLIENT LIST
Updated,Fri Mar 31 15:33:31 2023
Common Name,Real Address,Bytes Received,Bytes Sent,Connected Since
client,192.168.56.1:1194,3019,3260,Fri Mar 31 15:33:24 2023
ROUTING TABLE
Virtual Address,Common Name,Real Address,Last Ref
10.10.10.6,client,192.168.56.1:1194,Fri Mar 31 15:33:24 2023
GLOBAL STATS
Max bcast/mcast queue length,0
END
```

Подключаемся к серверу с хоста и проверяем пинг по внутреннему IP сервера в туннеле:

```bash
root@tw4-mint:/etc/openvpn/client# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s31f6: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 84:7b:eb:1d:18:2b brd ff:ff:ff:ff:ff:ff
3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether e4:a4:71:d9:a8:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.88.56/24 brd 192.168.88.255 scope global dynamic noprefixroute wlp2s0
       valid_lft 350sec preferred_lft 350sec
    inet6 fe80::3e62:ae0b:d807:97a4/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:aa:7a:8c:65 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: vboxnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0a:00:27:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.1/24 brd 192.168.56.255 scope global vboxnet0
       valid_lft forever preferred_lft forever
    inet6 fe80::800:27ff:fe00:0/64 scope link 
       valid_lft forever preferred_lft forever
6: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.10.10.6 peer 10.10.10.5/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::37f4:69ff:c9cb:3df8/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
root@tw4-mint:/etc/openvpn/client#
root@tw4-mint:/etc/openvpn/client# ip r
default via 192.168.88.1 dev wlp2s0 proto dhcp metric 600 
10.10.10.0/24 via 10.10.10.5 dev tun0 
10.10.10.5 dev tun0 proto kernel scope link src 10.10.10.6 
169.254.0.0/16 dev wlp2s0 scope link metric 1000 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
192.168.56.0/24 via 10.10.10.5 dev tun0 
192.168.88.0/24 dev wlp2s0 proto kernel scope link src 192.168.88.56 metric 600        

```
