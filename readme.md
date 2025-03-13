# OTUS PRO Homework 35 VPN

## Домашняя работа 35: Мосты, туннели и VPN

### Домашнее задание:
Описание домашнего задания:

1. Подготовка рабочего места

2. Развернуть 3 виртуальные машины

3. Объединить их разными vlan
* Настроить OSPF между машинами на базе Quagga
* Изобразить ассиметричный роутинг
* Сделать один из линков "дорогим", но что бы при этом роутинг был симметричным 
---
### 1. Подготовка рабочего места:
Выполнение домашнего задания предполагает, что на компьютере установлен Vagrant+VirtualBox   
**[Как установить Vagrant на Debian 12](https://github.com/avlikh/Install_Vagrant_Debian12/blob/main/README.md)**   

Развернем Vagrant-стенд:
  - Создайте папку с проектом и зайдите в нее (например: /opt/otus/ospf):
```
mkdir -p /opt/otus/ospf ; cd /opt/otus/ospf
```
  - Клонируете проект с Github, набрав команду:
```
apt update -y && apt install git -y ; git clone https://github.com/avlikh/Otus_pro_35.git .
```
  - Запустите проект из папки, в которую склонировали проект (в нашем примере /opt/otus/ospf):
```
vagrant up
```
Результатом выполнения команды vagrant up станет созданные и настроенные 3 виртуальные машины:    
    
* **router1** 
* **router2** 
* **router3**   

---


## Выполнение задания:
### 2. Развернуть 3 виртуальные машины:    
    
В результате подготовки рабочего места (пункт 1) были созданы 3 виртуальные машины, к которым были применены необходимые настройки.    
Был выполнен playbook Ansible https://github.com/avlikh/Otus_pro_33/blob/main/provision.yml.    
Созданные виртуальные машины соединены между сосбой, согласно схеме сети (см. ниже)

---

### 3. Объединить их разными vlan:
* **3.1 Настроить OSPF между машинами на базе Quagga;**
* **3.2 изобразить ассиметричный роутинг;**
* **3.3 сделать один из линков "дорогим", но что бы при этом роутинг был симметричным.**
     
    
**Для начала нарисуем схему сети:**

![image](https://github.com/user-attachments/assets/33a02588-7054-4930-bdc5-cff49e2cc41e)
    
**3.1 настроить OSPF между машинами на базе Quagga;**    
     
Проверим работу нашего стенда.    

Для этого, подключимся к **router1**.    
`vagrant ssh router1`    
`sudo -i`    
    
Проверим что служба frr запущена:

```
Systemctl status frr
```

<details>
<summary> результат выполнения команды </summary>

```
● frr.service - FRRouting
     Loaded: loaded (/lib/systemd/system/frr.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-03-12 11:51:13 UTC; 33min ago
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 5732 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 5750 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 1117)
     Memory: 20.8M
     CGroup: /system.slice/frr.service
             ├─5750 /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd ospfd staticd
             ├─5761 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─5766 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─5768 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─5771 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Mar 12 11:51:08 router1 ospfd[5768]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Mar 12 11:51:08 router1 frrinit.sh[5778]: line 53: % Unknown command[4]: default-information originate always[5778|ospfd] Configuration file[/etc/frr/frr.conf] processing failure: 2
Mar 12 11:51:08 router1 watchfrr[5750]: [ZJW5C-1EHNT] restart all process 5751 exited with non-zero status 2
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Mar 12 11:51:13 router1 watchfrr[5750]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Mar 12 11:51:13 router1 frrinit.sh[5732]:  * Started watchfrr
Mar 12 11:51:13 router1 systemd[1]: Started FRRouting.
```
</details>

Видим, что служба frr запущена.    
    
<details>
<summary> Проверим доступность сетей: </summary>

```
root@router1:~# ping -c2 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=0.017 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=0.091 ms

--- 192.168.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1030ms
rtt min/avg/max/mdev = 0.017/0.054/0.091/0.037 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=63 time=2.57 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=63 time=2.80 ms

--- 192.168.20.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 2.565/2.680/2.796/0.115 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=0.610 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=1.45 ms

--- 192.168.30.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1030ms
rtt min/avg/max/mdev = 0.610/1.032/1.454/0.422 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 10.0.10.1
PING 10.0.10.1 (10.0.10.1) 56(84) bytes of data.
64 bytes from 10.0.10.1: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 10.0.10.1: icmp_seq=2 ttl=64 time=0.092 ms

--- 10.0.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1024ms
rtt min/avg/max/mdev = 0.020/0.056/0.092/0.036 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 10.0.11.1
PING 10.0.11.1 (10.0.11.1) 56(84) bytes of data.
64 bytes from 10.0.11.1: icmp_seq=1 ttl=64 time=0.530 ms
64 bytes from 10.0.11.1: icmp_seq=2 ttl=64 time=1.59 ms

--- 10.0.11.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1031ms
rtt min/avg/max/mdev = 0.530/1.060/1.591/0.530 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ping -c2 10.0.12.1
PING 10.0.12.1 (10.0.12.1) 56(84) bytes of data.
64 bytes from 10.0.12.1: icmp_seq=1 ttl=64 time=0.019 ms
64 bytes from 10.0.12.1: icmp_seq=2 ttl=64 time=0.095 ms

--- 10.0.12.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.019/0.057/0.095/0.038 ms
```
</details>

Видим, что все сети доступны.
        
    

<details>
<summary> Посмотрим маршруты до 192.168.30.0/24 сети с включенным и выключенным интерфейсом enp0s9 </summary>

```
root@router1:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  192.168.30.1 (192.168.30.1)  0.457 ms  0.412 ms  0.388 ms
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# ifconfig enp0s9 down
root@router1:~# ^C
root@router1:~# ^C
root@router1:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  10.0.10.2 (10.0.10.2)  0.488 ms  0.447 ms  0.425 ms
 2  192.168.30.1 (192.168.30.1)  0.861 ms  0.976 ms  0.782 ms
```
</details>
      
Видим, что при отключении интерфейса enp0s9, маршрут перестраивается.      

<details>
<summary> Посмотрим таблицу маршрутизации роутера </summary>

```
root@router1:~# ifconfig enp0s9 up
root@router1:~# vtysh

Hello, this is FRRouting (version 10.2.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router1# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/1000] is directly connected, enp0s8, weight 1, 00:47:52
O>* 10.0.11.0/30 [110/200] via 10.0.12.2, enp0s9, weight 1, 00:00:21
O   10.0.12.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:00:21
O   192.168.10.0/24 [110/100] is directly connected, enp0s10, weight 1, 00:48:22
O>* 192.168.20.0/24 [110/300] via 10.0.12.2, enp0s9, weight 1, 00:00:21
O>* 192.168.30.0/24 [110/200] via 10.0.12.2, enp0s9, weight 1, 00:00:21
```
</details>

Видим, что все маршруты построены корректно.    
    
**Повторим те же действия на хосте router2**

Для этого, подключимся к **router2**.    
`vagrant ssh router2`    
`sudo -i`    
    
Проверим что служба frr запущена:

```
systemctl status frr
```

<details>
<summary> результат выполнения команды </summary>

```
● frr.service - FRRouting
     Loaded: loaded (/lib/systemd/system/frr.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-03-12 11:51:14 UTC; 56min ago
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 5658 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 5676 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 1117)
     Memory: 20.9M
     CGroup: /system.slice/frr.service
             ├─5676 /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd ospfd staticd
             ├─5687 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─5692 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─5694 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─5697 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Mar 12 11:51:09 router2 frrinit.sh[5704]: line 53: % Unknown command[4]: default-information originate always[5704|ospfd] Configuration file[/etc/frr/frr.conf] processing failure: 2
Mar 12 11:51:09 router2 watchfrr[5676]: [ZJW5C-1EHNT] restart all process 5677 exited with non-zero status 2
Mar 12 11:51:14 router2 watchfrr[5676]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Mar 12 11:51:14 router2 watchfrr[5676]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Mar 12 11:51:14 router2 watchfrr[5676]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Mar 12 11:51:14 router2 watchfrr[5676]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Mar 12 11:51:14 router2 watchfrr[5676]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Mar 12 11:51:14 router2 frrinit.sh[5658]:  * Started watchfrr
Mar 12 11:51:14 router2 systemd[1]: Started FRRouting.
Mar 12 11:51:39 router2 ospfd[5694]: [S5PCG-77H23] Packet[DD]: Neighbor 192.168.50.10 Negotiation done (Master).
```
</details>

Видим, что служба frr запущена.    
    
<details>
<summary> Проверим доступность сетей: </summary>

```
root@router2:~# ping -c2 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=63 time=2.86 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=63 time=3.14 ms

--- 192.168.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 2.864/3.001/3.138/0.137 ms
root@router2:~# ^C
root@router2:~# ^C
root@router2:~# ping -c2 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=0.019 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=0.091 ms

--- 192.168.20.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1456ms
rtt min/avg/max/mdev = 0.019/0.055/0.091/0.036 ms
root@router2:~# ^C
root@router2:~# ^C
root@router2:~# ping -c2 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=1.21 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=1.04 ms

--- 192.168.30.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.043/1.128/1.214/0.085 ms
root@router2:~# ^C
root@router2:~# ^C
root@router2:~# ping -c2 10.0.10.1
PING 10.0.10.1 (10.0.10.1) 56(84) bytes of data.
64 bytes from 10.0.10.1: icmp_seq=1 ttl=64 time=0.690 ms
64 bytes from 10.0.10.1: icmp_seq=2 ttl=64 time=1.39 ms

--- 10.0.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1014ms
rtt min/avg/max/mdev = 0.690/1.039/1.388/0.349 ms
root@router2:~# ^C
root@router2:~# ^C
root@router2:~# ping -c2 10.0.11.1
PING 10.0.11.1 (10.0.11.1) 56(84) bytes of data.
64 bytes from 10.0.11.1: icmp_seq=1 ttl=64 time=1.41 ms
64 bytes from 10.0.11.1: icmp_seq=2 ttl=64 time=1.54 ms

--- 10.0.11.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1004ms
rtt min/avg/max/mdev = 1.406/1.473/1.540/0.067 ms
root@router2:~# ^C
root@router2:~# ^C
root@router2:~# ping -c2 10.0.12.1
PING 10.0.12.1 (10.0.12.1) 56(84) bytes of data.
64 bytes from 10.0.12.1: icmp_seq=1 ttl=63 time=1.08 ms
64 bytes from 10.0.12.1: icmp_seq=2 ttl=63 time=2.74 ms

--- 10.0.12.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1129ms
rtt min/avg/max/mdev = 1.080/1.907/2.735/0.827 ms
```
</details>

Видим, что все сети доступны.
        
    

<details>
<summary> Посмотрим маршруты до 192.168.30.0/24 сети с включенным и выключенным интерфейсом enp0s9 </summary>

```
root@router2:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  192.168.30.1 (192.168.30.1)  1.246 ms  1.191 ms  1.118 ms
root@router2:~# ^C
root@router2:~# ^C
root@router2:~# ifconfig enp0s9 down
root@router2:~# ^C
root@router2:~# ^C
root@router2:~# traceroute 192.168.30.1
traceroute to 192.168.30.1 (192.168.30.1), 30 hops max, 60 byte packets
 1  10.0.10.1 (10.0.10.1)  0.985 ms  0.890 ms  0.843 ms
 2  192.168.30.1 (192.168.30.1)  2.420 ms  2.613 ms  2.536 ms
```
</details>
      
Видим, что при отключении интерфейса enp0s9, маршрут перестраивается.      

<details>
<summary> Посмотрим таблицу маршрутизации роутера </summary>

```
root@router2:~# ifconfig enp0s9 up
root@router2:~# vtysh

Hello, this is FRRouting (version 10.2.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router2# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O   10.0.10.0/30 [110/1000] is directly connected, enp0s8, weight 1, 01:01:25
O   10.0.11.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:00:02
O>* 10.0.12.0/30 [110/200] via 10.0.11.1, enp0s9, weight 1, 00:00:02
O>* 192.168.10.0/24 [110/300] via 10.0.11.1, enp0s9, weight 1, 00:00:02
O   192.168.20.0/24 [110/100] is directly connected, enp0s10, weight 1, 01:01:24
O>* 192.168.30.0/24 [110/200] via 10.0.11.1, enp0s9, weight 1, 00:00:02

```
</details>

Видим, что все маршруты построены корректно.    
    
**Повторим те же действия на хосте router3**
    
Для этого, подключимся к **router3**.    
`vagrant ssh router3`    
`sudo -i`    
    
Проверим что служба frr запущена:

```
systemctl status frr
```

<details>
<summary> результат выполнения команды </summary>

```
● frr.service - FRRouting
     Loaded: loaded (/lib/systemd/system/frr.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-03-12 11:51:13 UTC; 1h 4min ago
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 5567 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 5584 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 1117)
     Memory: 20.9M
     CGroup: /system.slice/frr.service
             ├─5584 /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd ospfd staticd
             ├─5595 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─5600 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─5602 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─5605 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Mar 12 11:51:13 router3 watchfrr[5584]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Mar 12 11:51:13 router3 watchfrr[5584]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Mar 12 11:51:13 router3 watchfrr[5584]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Mar 12 11:51:13 router3 watchfrr[5584]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Mar 12 11:51:13 router3 frrinit.sh[5567]:  * Started watchfrr
Mar 12 11:51:13 router3 systemd[1]: Started FRRouting.
Mar 12 11:51:38 router3 ospfd[5602]: [S5PCG-77H23] Packet[DD]: Neighbor 192.168.50.11 Negotiation done (Master).
Mar 12 11:51:43 router3 ospfd[5602]: [S5PCG-77H23] Packet[DD]: Neighbor 192.168.50.10 Negotiation done (Master).
Mar 12 12:39:09 router3 ospfd[5602]: [S5PCG-77H23] Packet[DD]: Neighbor 192.168.50.10 Negotiation done (Master).
Mar 12 12:52:31 router3 ospfd[5602]: [S5PCG-77H23] Packet[DD]: Neighbor 192.168.50.11 Negotiation done (Master).
```
</details>

Видим, что служба frr запущена.    
    
<details>
<summary> Проверим доступность сетей: </summary>

```
root@router3:~# ping -c2 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=0.500 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=1.44 ms

--- 192.168.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1091ms
rtt min/avg/max/mdev = 0.500/0.970/1.440/0.470 ms
root@router3:~# ^C
root@router3:~# ^C
root@router3:~# ping -c2 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=0.493 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=1.65 ms

--- 192.168.20.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1029ms
rtt min/avg/max/mdev = 0.493/1.072/1.652/0.579 ms
root@router3:~# ^C
root@router3:~# ^C
root@router3:~# ping -c2 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=0.018 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=0.155 ms

--- 192.168.30.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1011ms
rtt min/avg/max/mdev = 0.018/0.086/0.155/0.068 ms
root@router3:~# ^C
root@router3:~# ^C
root@router3:~# ping -c2 10.0.10.1
PING 10.0.10.1 (10.0.10.1) 56(84) bytes of data.
64 bytes from 10.0.10.1: icmp_seq=1 ttl=64 time=1.00 ms
64 bytes from 10.0.10.1: icmp_seq=2 ttl=64 time=2.51 ms

--- 10.0.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1025ms
rtt min/avg/max/mdev = 1.003/1.754/2.505/0.751 ms
root@router3:~# ^C
root@router3:~# ^C
root@router3:~# ping -c2 10.0.11.1
PING 10.0.11.1 (10.0.11.1) 56(84) bytes of data.
64 bytes from 10.0.11.1: icmp_seq=1 ttl=64 time=0.019 ms
64 bytes from 10.0.11.1: icmp_seq=2 ttl=64 time=0.104 ms

--- 10.0.11.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1116ms
rtt min/avg/max/mdev = 0.019/0.061/0.104/0.042 ms
root@router3:~# ^C
root@router3:~# ^C
root@router3:~# ping -c2 10.0.12.1
PING 10.0.12.1 (10.0.12.1) 56(84) bytes of data.
64 bytes from 10.0.12.1: icmp_seq=1 ttl=64 time=0.616 ms
64 bytes from 10.0.12.1: icmp_seq=2 ttl=64 time=1.63 ms

--- 10.0.12.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1034ms
rtt min/avg/max/mdev = 0.616/1.124/1.632/0.508 ms
```
</details>

Видим, что все сети доступны.
        
    

<details>
<summary> Посмотрим маршруты до 192.168.10.0/24 сети с включенным и выключенным интерфейсом enp0s9 </summary>

```
root@router3:~# traceroute 192.168.10.1
traceroute to 192.168.10.1 (192.168.10.1), 30 hops max, 60 byte packets
 1  192.168.10.1 (192.168.10.1)  0.821 ms  0.497 ms  0.437 ms
root@router3:~# ^C
root@router3:~# ^C
root@router3:~# ifconfig enp0s9 down
root@router3:~# traceroute 192.168.10.1
traceroute to 192.168.10.1 (192.168.10.1), 30 hops max, 60 byte packets
 1  10.0.11.2 (10.0.11.2)  0.894 ms  0.405 ms  0.552 ms
 2  192.168.10.1 (192.168.10.1)  0.811 ms  0.727 ms  0.626 ms
root@router3:~#
```
</details>
      
Видим, что при отключении интерфейса enp0s9, маршрут перестраивается.      

<details>
<summary> Посмотрим таблицу маршрутизации роутера </summary>

```
root@router3:~# ifconfig enp0s9 up
root@router3:~# vtysh

Hello, this is FRRouting (version 10.2.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

router3# show ip route ospf
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

O>* 10.0.10.0/30 [110/1100] via 10.0.11.2, enp0s8, weight 1, 00:00:08
  *                         via 10.0.12.1, enp0s9, weight 1, 00:00:08
O   10.0.11.0/30 [110/100] is directly connected, enp0s8, weight 1, 01:09:59
O   10.0.12.0/30 [110/100] is directly connected, enp0s9, weight 1, 00:00:12
O>* 192.168.10.0/24 [110/200] via 10.0.12.1, enp0s9, weight 1, 00:00:08
O>* 192.168.20.0/24 [110/200] via 10.0.11.2, enp0s8, weight 1, 00:08:32
O   192.168.30.0/24 [110/100] is directly connected, enp0s10, weight 1, 01:09:59
```
</details>

Видим, что все маршруты построены корректно.
 
---

**3.2 Изобразить ассиметричный роутинг**    
     
<details>
<summary>Добавим в Ansible-playbook (provision.yml) натройку ассиметричного роутинга</summary>

```
  - name: set up asynchronous routing
    sysctl:
      name: net.ipv4.conf.all.rp_filter
      value: '0'
      state: present
```
</details>
    
<details>
<summary>В файле шаблона (template/frr.conf.j2) в конец секции настроки интерфейса enp0s8, добавляем условие</summary>

```
{% if ansible_hostname == 'router1' %}
 ip ospf cost 1000
{% else %}
 !ip ospf cost 450
{% endif %}
```
</details>
    
Запустим Ansible playbook:     
```
ansible-playbook provision.yml
```     
     
**Далее, проверим работу маршрутов.**     
Для этого, зайдем в хост router1 и запустим ping с ip 192.168.10.1 до 192.168.20.1     
```
vagrant ssh router1
```
```
sudo -i
```

<details>
<summary>Запустим ping с ip 192.168.10.1 до 192.168.20.1</summary>

```
root@router1:~# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=63 time=0.954 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=63 time=1.94 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=63 time=2.28 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=63 time=2.39 ms
```
</details>
     
Далее, не прерывая ping, в отдельном окне терминала зайдем на хост **router2**
```
vagrant ssh router2
```
```
sudo -i
```
     
<details>
<summary>Запустим tcpdump на интерфейсе enp0s9</summary>

```
root@router2:~# tcpdump -i enp0s9
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
21:12:09.360184 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 5, length 64
21:12:10.370177 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 6, length 64
21:12:10.592426 ARP, Request who-has 10.0.11.1 tell router2, length 28
21:12:10.593912 ARP, Reply 10.0.11.1 is-at 08:00:27:f7:1b:a3 (oui Unknown), length 46
21:12:11.377218 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 7, length 64
21:12:12.505307 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 8, length 64
21:12:12.989033 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
21:12:12.990219 IP 10.0.11.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
21:12:13.514645 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 9, length 64
21:12:14.523638 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 10, length 64
21:12:15.549398 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 11, length 64
21:12:16.554549 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 12, length 64
21:12:17.559103 IP router2 > 192.168.10.1: ICMP echo reply, id 8, seq 13, length 64
^C
13 packets captured
13 packets received by filter
0 packets dropped by kernel
```
</details>     
     
<details>
<summary>Затем запустим tcpdump на интерфейсе enp0s8</summary>

```
root@router2:~# tcpdump -i enp0s8
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
21:14:29.858242 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 144, length 64
21:14:30.861891 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 145, length 64
21:14:31.866003 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 146, length 64
21:14:32.870738 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 147, length 64
21:14:33.017917 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
21:14:33.020396 IP 10.0.10.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
21:14:33.876070 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 148, length 64
21:14:34.895844 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 149, length 64
21:14:35.901491 IP 192.168.10.1 > router2: ICMP echo request, id 8, seq 150, length 64
21:14:36.104579 ARP, Request who-has router2 tell 10.0.10.1, length 46
21:14:36.104650 ARP, Reply router2 is-at 08:00:27:8c:0d:7d (oui Unknown), length 28
^C
11 packets captured
11 packets received by filter
0 packets dropped by kernel
```
</details>         
     
Видим, что интерфейс enp0s9 только получает трафик с адреса 192.168.10.1, а интерфейс enp0s8 - только отправляет трафик на адрес 192.168.10.1

**Итак, мы видим что ассиметричный роутинг работает**

---

**3.3 Сделать один из линков "дорогим", но что бы при этом роутинг был симметричным**
    
<summary>В файле шаблона (template/frr.conf.j2) изменим прошлое добавленное условие на новое</summary>

```
{% if ansible_hostname == 'router1' %}
 !ip ospf cost 1000
{% elif ansible_hostname == 'router2' and symmetric_routing == true %}
 !ip ospf cost 1000
{% else %}
 !ip ospf cost 450
{% endif %}
```
</details>
    
В файл переменных плейбука (defaults/main.yml) добавим переменную symmetric_routing. Для включения симметричного роутинга присвоим ей значение true

Чтобы не выполнять весь Ansuble-playbook запустим его с тегом setup_ospf, благодаря котоому будет выполнени только перенастройка и перезапуск FRR     
```
ansible-playbook provision.yml -t setup_ospf
```
     
**Далее, проверим работу маршрутов.**     
Для этого, зайдем в хост router1 и запустим ping с ip 192.168.10.1 до 192.168.20.1     
```
vagrant ssh router1
```
```
sudo -i
```

<details>
<summary>Запустим ping с ip 192.168.10.1 до 192.168.20.1</summary>

```
root@router1:~# ping -I 192.168.10.1 192.168.20.1
PING 192.168.20.1 (192.168.20.1) from 192.168.10.1 : 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=63 time=0.932 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=63 time=2.80 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=63 time=2.71 ms
```
</details>
     
Далее, не прерывая ping, в отдельном окне терминала зайдем на хост **router2**
```
vagrant ssh router2
```
```
sudo -i
```
     
<details>
<summary>Запустим tcpdump на интерфейсе enp0s9</summary>

```
root@router2:~# tcpdump -i enp0s9
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
21:37:09.389233 IP 192.168.10.1 > router2: ICMP echo request, id 10, seq 23, length 64
21:37:09.389331 IP router2 > 192.168.10.1: ICMP echo reply, id 10, seq 23, length 64
21:37:09.957299 IP 10.0.11.1 > ospf-all.mcast.net: OSPFv2, Hello, length 48
21:37:09.957959 IP router2 > ospf-all.mcast.net: OSPFv2, Hello, length 48
21:37:10.392994 IP 192.168.10.1 > router2: ICMP echo request, id 10, seq 24, length 64
21:37:10.393035 IP router2 > 192.168.10.1: ICMP echo reply, id 10, seq 24, length 64
21:37:11.399348 IP 192.168.10.1 > router2: ICMP echo request, id 10, seq 25, length 64
21:37:11.399448 IP router2 > 192.168.10.1: ICMP echo reply, id 10, seq 25, length 64
21:37:12.402806 IP 192.168.10.1 > router2: ICMP echo request, id 10, seq 26, length 64
21:37:12.402907 IP router2 > 192.168.10.1: ICMP echo reply, id 10, seq 26, length 64
21:37:13.413626 IP 192.168.10.1 > router2: ICMP echo request, id 10, seq 27, length 64
21:37:13.413725 IP router2 > 192.168.10.1: ICMP echo reply, id 10, seq 27, length 64
21:37:14.440880 IP 192.168.10.1 > router2: ICMP echo request, id 10, seq 28, length 64
21:37:14.440979 IP router2 > 192.168.10.1: ICMP echo reply, id 10, seq 28, length 64
21:37:15.452623 IP 192.168.10.1 > router2: ICMP echo request, id 10, seq 29, length 64
21:37:15.452727 IP router2 > 192.168.10.1: ICMP echo reply, id 10, seq 29, length 64
^C
16 packets captured
16 packets received by filter
0 packets dropped by kernel
```
</details>     
     
Видим, что интерфейс enp0s9 и получает трафик с адреса 192.168.10.1, и отправляет трафик на него же.
     
вывод: трафик между роутерами ходит симметрично.


