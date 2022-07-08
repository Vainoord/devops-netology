## 3.4. Операционные системы, лекция 2
---
1. >Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:

Установка node_exporter проведена по мануалу из ссылки: https://devopscube.com/monitor-linux-servers-prometheus-node-exporter/
В Vagrantconfig добавлен проброс порта на гостевую машину:
```
config.vm.network "forwarded_port", guest: 9100, host: 9100
```
Unit-файл Node Exporter:
```
vagrant@dev-vm:~$ cat /etc/systemd/system/node_exporter.service 
[Unit]
Description=Node Exporter
After=network.target
 
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter $LOGGING $WEB_SOCKET $CPU_INFO
EnvironentFile=/etc/default/node_exporter
 
[Install]
WantedBy=multi-user.target

```
* >поместите его в автозагрузку
```
vagrant@dev-vm:~$ sudo systemctl --all list-unit-files --type=service

UNIT FILE                                  STATE           VENDOR PRESET
...
networkd-dispatcher.service                enabled         enabled      
node_exporter.service                      enabled         enabled      
ondemand.service                           enabled         enabled   
...
```
* >предусмотрите возможность добавления опций к запускаемому процессу через внешний файл

В файле `/etc/systemd/system/node_exporter.service`:
```
[Service]
ExecStart=/usr/local/bin/node_exporter $LOGGING $WEB_SOCKET $CPU_INFO
EnvironentFile=/etc/default/node_exporter
```
Файл `/etc/default/node_exporter`:
```
vagrant@dev-vm:~$ cat /etc/default/node_exporter 
WEB_SOCKET="--web.listen-address=":9100""
LOGGING="--log.level=info"
CPU_INFO="--collector.cpu.info"
VERSION="--version"
```
Результат добавления опций можно посмотреть в файле environ процесса node_exporter:
```
vagrant@dev-vm:~$ sudo cat /proc/2847/environ
LANG=en_US.UTF-8PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/binHOME=/home/node_exporterLOGNAME=node_exporterUSER=node_exporterINVOCATION_ID=54960b0fb82b44bd8ae5865e47faf9abJOURNAL_STREAM=9:37603WEB_SOCKET=--web.listen-address=":9100"LOGGING=--log.level=infoCPU_INFO=--collector.cpu.infoVERSION=--version
```
* >удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается
Чтобы убедиться, что модуль запускается и добавить его в автозапуск достаточно выполнить следующую серию команд:
```
vagrant@dev-vm:~$ sudo systemctl daemon-reload

vagrant@dev-vm:~$ sudo systemctl start node_exporter

vagrant@dev-vm:~$ systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-07-06 11:04:55 UTC; 2h 1min ago
   Main PID: 2847 (node_exporter)
      Tasks: 4 (limit: 2274)
     Memory: 2.3M
     CGroup: /system.slice/node_exporter.service
             └─2847 /usr/local/bin/node_exporter --log.level=info --web.listen-address=:9100 --collector.cp>

vagrant@dev-vm:~$ sudo systemctl enable node_exporter
```
После перезагрузки VM автозапуск процесса выполняется в штатном режиме.

---
2. >Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
#### CPU: количество секунд, затраченных ЦП в каждом режиме для каждого ядра
- node_cpu_seconds_total{cpu="0",mode="idle"} 2696.79
- node_cpu_seconds_total{cpu="0",mode="iowait"} 1.64
- node_cpu_seconds_total{cpu="0",mode="irq"} 0
- node_cpu_seconds_total{cpu="0",mode="nice"} 0
- node_cpu_seconds_total{cpu="0",mode="softirq"} 1.14
- node_cpu_seconds_total{cpu="0",mode="steal"} 0
- node_cpu_seconds_total{cpu="0",mode="system"} 4.82
- node_cpu_seconds_total{cpu="0",mode="user"} 2.66

#### RAM: доступный и общий объем RAM и SWAP
- node_memory_MemAvailable_bytes 1.759883264e+09
- node_memory_MemFree_bytes 1.092694016e+09
- node_memory_MemTotal_bytes 2.079461376e+09
- node_memory_SwapCached_bytes 0
- node_memory_SwapFree_bytes 2.047864832e+09
- node_memory_SwapTotal_bytes 2.047864832e+09

#### DISK: записанное/прочитанное количество байт и количество этих итераций для каждого диска 
- node_disk_written_bytes_total{device="sda"} 9.396224e+07
- node_disk_writes_completed_total{device="sda"} 4410
- node_disk_reads_completed_total{device="sda"} 17518
- node_disk_read_bytes_total{device="sda"} 5.67432192e+08

#### NETWORK: количество переданных/полученных байт, пакетов и сколько было ошибок в приеме и передаче пакетов для каждого сетевого адаптера
- node_network_transmit_bytes_total{device="eth0"} 345174
- node_network_transmit_packets_total{device="eth0"} 3460
- node_network_transmit_errs_total{device="eth0"} 0
- node_network_receive_bytes_total{device="eth0"} 3.1194863e+07
- node_network_receive_packets_total{device="eth0"} 24526
- node_network_receive_errs_total{device="eth0"} 0

---
3. >Установите в свою виртуальную машину Netdata.

Netdata установлен:
```
vagrant@dev-vm:~$ systemctl status netdata
● netdata.service - netdata - Real-time performance monitoring
     Loaded: loaded (/lib/systemd/system/netdata.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-07-07 09:10:35 UTC; 3min 50s ago
       Docs: man:netdata
             file:///usr/share/doc/netdata/html/index.html
             https://github.com/netdata/netdata
   Main PID: 653 (netdata)
      Tasks: 22 (limit: 2274)
     Memory: 69.1M
     CGroup: /system.slice/netdata.service
             ├─653 /usr/sbin/netdata -D
             ├─800 /usr/lib/netdata/plugins.d/nfacct.plugin 1
             ├─805 /usr/lib/netdata/plugins.d/apps.plugin 1
             └─814 bash /usr/lib/netdata/plugins.d/tc-qos-helper.sh 1

Jul 07 09:10:35 dev-vm systemd[1]: Started netdata - Real-time performance monitoring.
Jul 07 09:10:36 dev-vm netdata[653]: SIGNAL: Not enabling reaper
Jul 07 09:10:36 dev-vm netdata[653]: 2022-07-07 09:10:36: netdata INFO  : MAIN : SIGNAL: Not enabling reaper
vagrant@dev-vm:~$ 

```
Сделан проброс порта `config.vm.network "forwarded_port", guest: 19999, host: 19999`.
```
vagrant@dev-vm:~$ sudo lsof -i :19999
COMMAND PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
netdata 653 netdata    4u  IPv4  23502      0t0  TCP *:19999 (LISTEN)
netdata 653 netdata   47u  IPv4  31828      0t0  TCP dev-vm:19999->_gateway:65106 (ESTABLISHED)
netdata 653 netdata   49u  IPv4  33068      0t0  TCP dev-vm:19999->_gateway:65107 (ESTABLISHED)
netdata 653 netdata   50u  IPv4  33073      0t0  TCP dev-vm:19999->_gateway:65111 (ESTABLISHED)
netdata 653 netdata   51u  IPv4  31830      0t0  TCP dev-vm:19999->_gateway:65131 (ESTABLISHED)
netdata 653 netdata   52u  IPv4  33071      0t0  TCP dev-vm:19999->_gateway:65109 (ESTABLISHED)
netdata 653 netdata   53u  IPv4  31832      0t0  TCP dev-vm:19999->_gateway:65134 (ESTABLISHED)
```
Полный перечень метрик и плагинов netdata можно посмотреть в http://localhost:19999/netdata.conf

---
4. >Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

Да, в логах можно найти информацию, что машина запущена в виртуальной среде:
```
vagrant@dev-vm:/etc/netdata$ sudo dmesg | grep "Hypervisor detected"
[    0.000000] Hypervisor detected: KVM
vagrant@dev-vm:/etc/netdata$ dmesg | grep virtual
[    0.002462] CPU MTRRs all blank - virtualized system.
[    0.119719] Booting paravirtualized kernel on KVM
[    4.032810] systemd[1]: Detected virtualization oracle.
```
Можно воспользоваться утилитой dmidecode для получения информации об ОС:
```
vagrant@dev-vm:/etc/netdata$ sudo dmidecode -t system
# dmidecode 3.2
Getting SMBIOS data from sysfs.
SMBIOS 2.5 present.

Handle 0x0001, DMI type 1, 27 bytes
System Information
	Manufacturer: innotek GmbH
	Product Name: VirtualBox
	Version: 1.2
	Serial Number: 0
	UUID: 35bf98c3-8179-a441-81fc-a41b1ebbca26
	Wake-up Type: Power Switch
	SKU Number: Not Specified
	Family: Virtual Machine
```
----
5. >Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр.

Из `man proc`:
```
/proc/sys/fs/nr_open (since Linux 2.6.25)
              This  file  imposes  ceiling on the value to which the RLIMIT_NOFILE resource limit can be
              raised (see getrlimit(2)).  This ceiling is enforced for both unprivileged and  privileged
              process.   The  default  value in this file is 1048576.  (Before Linux 2.6.25, the ceiling
              for RLIMIT_NOFILE was hard-coded to the same value.)
```
Из `man getrlimit`:
```
RLIMIT_NOFILE
              This specifies a value one greater than the maximum file descriptor  number  that  can  be
              opened  by  this process.
```
fs.nr_open - максимальное количество файловых дескрипторов на один процесс.
Число по умолчанию = 1048576. Это число устанавливается и для привилегированных (принадлежащих  root) процессов, и для непривилегированных.
```
vagrant@dev-vm:/etc/netdata$ sysctl -n fs.nr_open
1048576
```
У процесса с PID=1 указано такое количество файловых дескрипторов:
```
vagrant@dev-vm:/etc/netdata$ cat /proc/1/limits | grep 'open files'
Max open files            1048576              1048576              files  
```
Общий максимальный количественный лимит для ОС на все процессы:
```
vagrant@dev-vm:/etc/netdata$ cat /proc/sys/fs/file-max
9223372036854775807
```
>Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?

Процессы имеют так называемые soft limit и hard limit на использование файловых дескрипторов (Max open files). Если процесс запущен обычным пользователем, то количество файловых дескрипторов устанавливается по умолчанию из soft limit и не превышает hard limit. Только root пользователь может менять значение hard limit у процессов.
soft limit <= hard limit <= fs.nr_open 

---
6. >Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.

Запустим процесс sleep под root - `unshare -f -p --mount-proc sleep 1h`.
В пользовательском пространстве имен он выглядит так:
```
vagrant@dev-vm:~$ ps aux | grep sleep
root        1640  0.0  0.0   7232   584 pts/0    S+   11:02   0:00 unshare -f -p --mount-proc sleep 1h
root        1641  0.0  0.0   7228   516 pts/0    S+   11:02   0:00 sleep 1h
```
Переключимся в namespace этого процесса и посмотрим его PID:
```
vagrant@dev-vm:~$ sudo nsenter --target 1641 --pid --mount

root@dev-vm:/# ps aux | grep sleep
root           1  0.0  0.0   7228   516 pts/0    S+   11:02   0:00 sleep 1h
root          26  0.0  0.0   8160   720 pts/1    S+   11:08   0:00 grep --color=auto sleep

root@dev-vm:/# pstree -p
sleep(1)
```

---
7. >Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (это важно, поведение в других ОС не проверялось). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов dmesg расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

`:(){ :|:& };:` - это bash функция, другое ее название - "fork bomb". Что-то похожее на форму DoS атаки на Linux или Unix-based системы. Функция выполняется рекурсивно.

1) Определяем функцию `:` без параметров - `:()`
2) В блоке `{ }` функции она вызывает сама себя и направит вывод в вызов другой функции `:`
3) `&` помещает вызов функции в фон, таким образом эта функция не завершается и продолжает занимать ресурсы.
4) `;` завершает определение функции
5) `:` это вызов функции
``` 
:()
{
 :|:&
};:
```
Таким образом, bash фунция порождает в геометрической последовательности своих потомков и быстро расходует лимит одновременно запущенных процессов.
Через dmesg видно, что на каком-то этапе было заблокировано создание новых процессов (fork rejected):
```
[Fri Jul  8 08:55:39 2022] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-1.scope
```
Ulimit поможет установить ограничение на количество запущенных процессов для пользователя.
```
ulimit: ulimit [-SHabcdefiklmnpqrstuvxPT] [limit]
    Modify shell resource limits.
    
    Provides control over the resources available to the shell and processes
    it creates, on systems that allow such control.
    
    Options:
      -S	use the `soft' resource limit
      -H	use the `hard' resource limit
      -a	all current limits are reported
      -b	the socket buffer size
      -c	the maximum size of core files created
      -d	the maximum size of a process's data segment
      -e	the maximum scheduling priority (`nice')
      -f	the maximum size of files written by the shell and its children
      -i	the maximum number of pending signals
      -k	the maximum number of kqueues allocated for this process
      -l	the maximum size a process may lock into memory
      -m	the maximum resident set size
      -n	the maximum number of open file descriptors
      -p	the pipe buffer size
      -q	the maximum number of bytes in POSIX message queues
      -r	the maximum real-time scheduling priority
      -s	the maximum stack size
      -t	the maximum amount of cpu time in seconds
      -u	the maximum number of user processes
      -v	the size of virtual memory
      -x	the maximum number of file locks
      -P	the maximum number of pseudoterminals
      -T	the maximum number of threads
```
Для измения числа процессов в сессии пользователя можно нужно установить ulimit с флагом `-u`.
По умолчанию, максимальное число процессов для пользователя в VM = 7580
```
vagrant@dev-vm:~$ ulimit -u
7580
```
---
