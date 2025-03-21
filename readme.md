# OTUS PRO Homework 35 VPN

## Домашняя работа 35: Мосты, туннели и VPN

### Домашнее задание:
Описание домашнего задания:

1. Подготовка рабочего места

2. Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях

3. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ 
---
### 1. Подготовка рабочего места:
Выполнение домашнего задания предполагает, что на компьютере установлен Vagrant+VirtualBox   
**[Как установить Vagrant на Debian 12](https://github.com/avlikh/Install_Vagrant_Debian12/blob/main/README.md)**   

Развернем Vagrant-стенд:
  - Создайте папку с проектом и зайдите в нее (например: /opt/otus/vpn):
```
mkdir -p /opt/otus/vpn ; cd /opt/otus/vpn
```
  - Клонируете проект с Github, набрав команду:
```
apt update -y && apt install git -y ; git clone https://github.com/avlikh/Otus_pro_35.git .
```
---

## Выполнение задания:
### 2. Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях    
    
* Запустим vagrant-файл для стенда tun/tup. Стенд находится в папке tun-tap (в нашем примере /opt/otus/vpn/tun-tap):
```
cd tun-tap
vagrant up
```

**2.1 Далее развернем стенд TAP VPN**    
     
**Запустим Ansible-playbook vpn_tap.yaml:**
     
```
ansible-playbook vpn_tap.yaml
```
<details>
<summary> результат выполнения команды </summary>

```
root@deb4likh:/opt/otus/vpn/tun-tap# ansible-playbook vpn_tap.yaml

PLAY [OpenVPN server setup] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [client]
ok: [server]

TASK [Install packages] *************************************************************************************************************************************************************************************************
changed: [server]
changed: [client]

PLAY [OpenVPN server setup] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [server]

TASK [Static key generation] ********************************************************************************************************************************************************************************************
changed: [server]

TASK [Send static key to managed host] **********************************************************************************************************************************************************************************
changed: [server]

TASK [Generate OpenVpn config] ******************************************************************************************************************************************************************************************
changed: [server]

TASK [Generate OpenVpn service] *****************************************************************************************************************************************************************************************
changed: [server]

TASK [Daemon reload] ****************************************************************************************************************************************************************************************************
ok: [server]

TASK [OpenVPN service start] ********************************************************************************************************************************************************************************************
changed: [server]

PLAY [OpenVPN client setup] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [client]

TASK [Copy static.key from management host] *****************************************************************************************************************************************************************************
changed: [client]

TASK [Geneate openvpn config] *******************************************************************************************************************************************************************************************
changed: [client]

TASK [Generate OpenVpn service] *****************************************************************************************************************************************************************************************
changed: [client]

TASK [Daemon reload] ****************************************************************************************************************************************************************************************************
ok: [client]

TASK [OpenVPN service start] ********************************************************************************************************************************************************************************************
changed: [client]

PLAY RECAP **************************************************************************************************************************************************************************************************************
client                     : ok=8    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
server                     : ok=9    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
</details>

**Замерим скорость в туннеле.**    
     

Для этого, зайдем на сервер "server" и запустим iperf в режиме сервера:
```
vagrant ssh server
```
```
sudo -i
```
```
iperf3 -s &
```
<details>
<summary> результат выполнения команды </summary>

```
root@server:~# iperf3 -s &
[1] 3566
root@server:~# -----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------

```
</details>

Не выходя из "server", в отдельном окне терминала зайдем на сервер "client" и запустим iperf в режиме клиента:
```
vagrant ssh server
```
```
sudo -i
```
```
iperf3 -c 10.10.10.1 -t 40 -i 5
```
<details>
<summary> результат выполнения команды </summary>

```
vagrant@client:~$ iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 44846 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  70.5 MBytes   118 Mbits/sec   51   1.17 MBytes
[  5]   5.00-10.00  sec  70.0 MBytes   117 Mbits/sec   70    743 KBytes
[  5]  10.00-15.01  sec  68.8 MBytes   115 Mbits/sec    0    759 KBytes
[  5]  15.01-20.00  sec  72.5 MBytes   122 Mbits/sec    0    948 KBytes
[  5]  20.00-25.00  sec  70.0 MBytes   117 Mbits/sec  158    360 KBytes
[  5]  25.00-30.00  sec  68.8 MBytes   115 Mbits/sec    0    476 KBytes
[  5]  30.00-35.01  sec  70.0 MBytes   117 Mbits/sec    0    570 KBytes
[  5]  35.01-40.01  sec  61.2 MBytes   103 Mbits/sec   19    909 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.01  sec   552 MBytes   116 Mbits/sec  298             sender
[  5]   0.00-40.27  sec   552 MBytes   115 Mbits/sec                  receiver

iperf Done.
```
</details>

Видим, что средняя скорость TAP = **116 Mbit/sec**

---

**2.2 Далее развернем стенд TUN VPN**    
     
**Запустим Ansible-playbook vpn_tun.yaml:**
     
```
ansible-playbook vpn_tun.yaml
```
<details>
<summary> результат выполнения команды </summary>

```
root@deb4likh:/opt/otus/vpn/tun-tap# ansible-playbook vpn_tun.yaml

PLAY [OpenVPN server setup] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [server]

TASK [Generate OpenVPN server sonfig] ***********************************************************************************************************************************************************************************
changed: [server]

TASK [Daemon reload] ****************************************************************************************************************************************************************************************************
ok: [server]

TASK [OpenVPN server restart] *******************************************************************************************************************************************************************************************
changed: [server]

PLAY [OpenVPN client setup] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [client]

TASK [Generate OpenVpn config] ******************************************************************************************************************************************************************************************
changed: [client]

TASK [Generate OpenVpn service] *****************************************************************************************************************************************************************************************
ok: [client]

TASK [Daemon reload] ****************************************************************************************************************************************************************************************************
ok: [client]

TASK [OpenVPN server restart] *******************************************************************************************************************************************************************************************
changed: [client]

PLAY RECAP **************************************************************************************************************************************************************************************************************
client                     : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
server                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
</details>

**Замерим скорость в туннеле.**    
     

Для этого, зайдем на сервер "server" и запустим iperf в режиме сервера:
```
vagrant ssh server
```
```
sudo -i
```
```
iperf3 -s &
```
<details>
<summary> результат выполнения команды </summary>

```
root@server:~# iperf3 -s &
[1] 1684
root@server:~# -----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------

```
</details>

Не выходя из "server", в отдельном окне терминала зайдем на сервер "client" и запустим iperf в режиме клиента:
```
vagrant ssh server
```
```
sudo -i
```
```
iperf3 -c 10.10.10.1 -t 40 -i 5
```
<details>
<summary> результат выполнения команды </summary>

```
root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 35642 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec  70.7 MBytes   118 Mbits/sec   85    361 KBytes
[  5]   5.00-10.00  sec  71.7 MBytes   120 Mbits/sec    2    392 KBytes
[  5]  10.00-15.00  sec  73.2 MBytes   123 Mbits/sec   24    334 KBytes
[  5]  15.00-20.00  sec  75.3 MBytes   126 Mbits/sec    8    266 KBytes
[  5]  20.00-25.00  sec  74.3 MBytes   125 Mbits/sec    3    354 KBytes
[  5]  25.00-30.00  sec  73.3 MBytes   123 Mbits/sec   18    361 KBytes
[  5]  30.00-35.00  sec  73.0 MBytes   122 Mbits/sec   35    207 KBytes
[  5]  35.00-40.00  sec  68.6 MBytes   115 Mbits/sec   21    296 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec   580 MBytes   122 Mbits/sec  196             sender
[  5]   0.00-40.04  sec   579 MBytes   121 Mbits/sec                  receiver

iperf Done.
```
</details>

Видим, что средняя скорость TUN = **122 Mbit/sec**

Получаем: 
* скорость в режиме **TAP**:  **116 Mbit/sec**
* скорость в режиме **TUN**:  **122 Mbit/sec**
    
**Подведем итог:**    
Исходя из замеров скорости видим что **TUN** немного быстрее **TAP**.    
    
В этом нет ничего удивительного т.к. **TAP** работает на L3 уровне (передает только IP пакеты), а **tap** работает на уровне L2.    
    
* **TUN** - Передает только IP-пакеты. Имеет меньше накладных расходов, так как не нужно обрабатывать Ethernet-кадры. Не передает широковещательные (broadcast) и мультимастинговые (multicast) пакеты.    
Данный режим может быть полезен для маршрутизации между сетями, с максимальной скоростью и низкой задержкой.    

* **TAP** - Передаются все Ethernet-кадры, включая ARP, что создает лишнюю нагрузку.    
Данный режим может быть полезен если важна полная эмуляция сети и поддержка нестандартных протоколов.




---

### 3. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ

* Запустим vagrant-файл для стенда tun/tup. Стенд находится в папке ras (в нашем примере /opt/otus/vpn/ras):
```
cd ras
vagrant up
```     
**Запустим Ansible-playbook vpn_tap.yaml:**
     
```
ansible-playbook ras_vpn.yaml
```
<details>
<summary> результат выполнения команды </summary>

```
root@deb4likh:/opt/otus/vpn/ras# ansible-playbook ras_vpn.yaml

PLAY [OpenVpn Server deploy] ********************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [ovpnserver]

TASK [openvpn : Install packages] ***************************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Create work dirs] ***************************************************************************************************************************************************************************************
changed: [ovpnserver] => (item=/etc/openvpn/pki)
changed: [ovpnserver] => (item=/etc/openvpn/client)

TASK [openvpn : Intit PKI] **********************************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Generate CA] ********************************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Generate server certificates] ***************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Generate client certificates] ***************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Generate openvpn server config] *************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Make client route] **************************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Generate openvpn service] *******************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Перезагрузка демона systemd] ****************************************************************************************************************************************************************************
ok: [ovpnserver]

TASK [openvpn : Enable and start systemd service] ***********************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Generate client config] *********************************************************************************************************************************************************************************
changed: [ovpnserver]

TASK [openvpn : Send keys and configs to managed host] ******************************************************************************************************************************************************************
changed: [ovpnserver] => (item={'src': '/etc/openvpn/pki/ca.crt', 'dest': '/opt/otus/vpn/ras/config/ca.crt'})
changed: [ovpnserver] => (item={'src': '/etc/openvpn/pki/issued/client.crt', 'dest': '/opt/otus/vpn/ras/config/client.crt'})
changed: [ovpnserver] => (item={'src': '/etc/openvpn/pki/private/client.key', 'dest': '/opt/otus/vpn/ras/config/client.key', 'mode': '0600'})
ok: [ovpnserver] => (item={'src': '/tmp/client.ovpn', 'dest': '/opt/otus/vpn/ras/config/client.ovpn'})

RUNNING HANDLER [openvpn : Restart OpenVPN] *****************************************************************************************************************************************************************************
changed: [ovpnserver]

PLAY RECAP **************************************************************************************************************************************************************************************************************
ovpnserver                 : ok=15   changed=13   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```
</details>    
     
Установим клиента OpenVPN на управляющую (хостовую) машину:
```
apt install openvpn
```
Зайдем в папку с ключами и конфигами OpenVPN:
```
cd config
```
* Примечание: в нашем примере, полный путь к папке будет: /opt/otus/vpn/ras/config
     
Запустим OpenVPN клиента в фоновом режиме:
```
sudo openvpn --config client.ovpn --daemon
```
    
Попробуем пингануть сервер OpenVPN:
```
ping -c2 10.10.10.1
```
<details>
<summary> результат выполнения команды </summary>

```
root@deb4likh:/opt/otus/vpn/ras/config# ping -c2 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.740 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=2.05 ms

--- 10.10.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1026ms
rtt min/avg/max/mdev = 0.740/1.393/2.047/0.653 ms

```
</details>
    
Видим что туннель работает.
