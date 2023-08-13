# Домашнее задание к занятию 1 «Disaster recovery и Keepalived»

------


### Задание 1
- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

------
### Решение:
У меня нет Cisco Packet Tracer, попробовал с эмулировать практическое задание в другой программе. 
Схема стенда: 

<img width="1066" alt="Screenshot 2023-08-13 at 20 10 41" src="https://github.com/otuzi/Disaster_recovery_Keepalived/assets/61628386/aeee7ba7-346a-4e0b-8e22-43d5d810a3b1">

Настройки маршрутизатора R1, с настроенным HSRP между R1 (Ethernet0/2) и R2 (Ethernet0/0): 
```
!
interface Ethernet0/0
 ip address 10.1.14.1 255.255.255.0
!
interface Ethernet0/1
 no ip address
!
interface Ethernet0/2
 ip address 10.1.123.1 255.255.255.0
 standby 1 ip 10.1.123.254
 standby 1 preempt
!
interface Ethernet0/3
 no ip address
!
!
router eigrp 1
 network 10.1.14.0 0.0.0.255
 network 10.1.123.0 0.0.0.255
 passive-interface Ethernet0/2
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
```
Настройки маршрутизатора R2 (Ethernet0/0): 
```
!
interface Ethernet0/0
 ip address 10.1.123.2 255.255.255.0
 standby 1 ip 10.1.123.254
 standby 1 priority 150
 standby 1 preempt
 standby 1 track 1 decrement 70
!
interface Ethernet0/1
 ip address 10.1.24.2 255.255.255.0
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
!
router eigrp 1
 network 10.1.24.0 0.0.0.255
 network 10.1.123.0 0.0.0.255
 passive-interface Ethernet0/0
!
ip forward-protocol nd
!         
!
no ip http server
no ip http secure-server
!
ip sla 1
 icmp-echo 10.1.24.4
 frequency 10
ip sla schedule 1 life forever start-time now
!
!
!
control-plane
```
Перестройка канала, при отключении инетерфейса: 

```
R2#configure terminal 
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#
R2(config)#int
R2(config)#interface eth
R2(config)#interface ethernet 0/1
R2(config-if)#
R2(config-if)#
R2(config-if)#shu
R2(config-if)#shutdown 
R2(config-if)#
R2(config-if)#
R2(config-if)#
*Aug 13 17:07:05.476: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.1.24.4 (Ethernet0/1) is down: i
nterface down
R2(config-if)#
*Aug 13 17:07:07.474: %LINK-5-CHANGED: Interface Ethernet0/1, changed state to administratively do
wn
*Aug 13 17:07:08.478: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/1, changed state t
o down
R2(config-if)#
*Aug 13 17:07:11.878: %TRACK-6-STATE: 1 ip sla 1 state Up -> Down
R2(config-if)#
*Aug 13 17:07:14.038: %HSRP-5-STATECHANGE: Ethernet0/0 Grp 1 state Active -> Speak
R2(config-if)#
*Aug 13 17:07:25.236: %HSRP-5-STATECHANGE: Ethernet0/0 Grp 1 state Speak -> Standby
R2(config-if)#
R2(config-if)#
R2(config-if)#
R2(config-if)#no shutdown           
R2(config-if)#
R2(config-if)#
R2(config-if)#
*Aug 13 17:07:57.603: %LINK-3-UPDOWN: Interface Ethernet0/1, changed state to up
*Aug 13 17:07:58.603: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/1, changed state t
o up
R2(config-if)#
*Aug 13 17:08:01.528: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.1.24.4 (Ethernet0/1) is up: new
 adjacency
R2(config-if)#
*Aug 13 17:08:06.899: %TRACK-6-STATE: 1 ip sla 1 state Down -> Up
R2(config-if)#
*Aug 13 17:08:09.027: %HSRP-5-STATECHANGE: Ethernet0/0 Grp 1 state Standby -> Active
R2(config-if)#
R2(config-if)#
```


### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

------
### Решение:
Настройка Keepalived с использованием скрипта для проверки доступности порта веб-сервера и наличия файла index.html:

```bash
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
}

vrrp_script check_webserver {
    script "/path/to/check_webserver.sh"
    interval 3
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100

    authentication {
        auth_type PASS
        auth_pass password
    }

    virtual_ipaddress {
        192.168.0.100
    }

    track_script {
        check_webserver
    }
}
```

В данном примере используется секция `vrrp_script`, где указывается путь к bash-скрипту `check_webserver.sh`, который будет выполняться каждые 3 секунды. В скрипте можно использовать команду `nc` для проверки доступности порта веб-сервера и команду `test` для проверки наличия файла index.html.

Скрипт `check_webserver.sh`:

```bash
#!/bin/bash

# Проверка доступности порта веб-сервера
nc -z localhost 80
result1=$?

# Проверка наличия файла index.html
test -f /var/www/html/index.html
result2=$?

# Если порт недоступен или файл отсутствует, возвращаем код отличный от нуля
if [ $result1 -ne 0 ] || [ $result2 -ne 0 ]; then
    exit 1
fi

exit 0
```

В данном скрипте сначала проверяется доступность порта веб-сервера с помощью команды `nc -z`, затем проверяется наличие файла index.html с помощью команды `test -f`. Если одно из условий не выполняется, скрипт возвращает код отличный от нуля.

В конфигурационном файле Keepalived нужно указать правильный путь к скрипту `check_webserver.sh`.
