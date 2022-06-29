## 3.3. Операционные системы, лекция 1
---
1. >Какой системный вызов делает команда `cd`?
```
vagrant@vagrant:~$ strace /bin/bash -c 'cd /tmp' 2>/home/vagrant/std
```
```
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")                           = 0
```
Системный вызов chdir, exit code = 0, команда завершена успешно

---
2. >Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.

БД `file` расположена в файле `/usr/lib/file/magic.mgc`. 
Обращение к БД проходит через `/usr/share/misc/magic.mgc`. Кроме этого, идет 
обращение к файлам `/etc/magic.mgc` (такого файла в ubuntu 20.04 нет) и `/etc/magic`.

```
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or 
directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0

openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
read(3, "# Magic local data for file(1) c"..., 4096) = 111
read(3, "", 4096)                       = 0
close(3)                                = 0

openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3

```
```
vagrant@vagrant:~$ file /usr/share/misc/magic.mgc 
/usr/share/misc/magic.mgc: symbolic link to ../../lib/file/magic.mgc
```
---
3. >Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

Предположим, есть файл в директории пользователя logfile.log . Будем писать в него результат выполнения команды `ping`:
```
vagrant@vagrant:~$ ping 8.8.8.8 > logfile.log
```
PID процесса - 1422
```
vagrant     1422  0.0  0.0   8844   924 pts/0    S+   14:17   0:00 ping 8.8.8.8
```
Размер файла увеличивается:
```
vagrant@vagrant:~$ sudo lsof -p 1422 | grep logfile
ping    1422 vagrant    1w   REG  253,0     4161 1323302 /home/vagrant/logfile.log
```
Удаляем файл и смотрим результат:
```
vagrant@vagrant:~$ rm logfile.log 
vagrant@vagrant:~$ sudo lsof -p 1422 | grep logfile
ping    1422 vagrant    1w   REG  253,0     8785 1323302 /home/vagrant/logfile.log (deleted)
```
Файл удален, запись продолжает идти. Процесс 1422 еще работает, данные пишутся в файловый дескриптор `/proc/1422/fd/1`.

3 варианта обнуления файла:
1) под учетной записью root: echo '' > /proc/1422/fd/1
2) скопировать `/dev/null` в дескриптор: sudo cp /dev/null /proc/1422/fd/1
3) уменьнить размер файла через команду `truncate`: sudo truncate -s 0 /proc/1422/fd/1

Но ни один из способов не сработал - файловый дескриптор продолжал расти в размерах.

---
4. >Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Зомби-процессы не занимают какие-либо ресурсы в ОС, но отображаются в 
таблице процессов с литерой Z.
Зомби-процессы нельзя убить SIGKILL командой, т.к. эти процесс уже в статусе dead. Можно воспользоваться kill -s SIGCHLD parent_pid, команда запустит у родительского процесса системный вызов wait(), чтобы подчистить дочерние зомби-процессы.
 
----
5. >В iovisor BCC есть утилита `opensnoop`:
```
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
```

>На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты?
```
vagrant@vagrant:~$ sudo opensnoop-bpfcc -d 1
PID    COMM               FD ERR PATH
1019   vminfo              6   0 /var/run/utmp
639    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
639    dbus-daemon        20   0 /usr/share/dbus-1/system-services
639    dbus-daemon        -1   2 /lib/dbus-1/system-services
639    dbus-daemon        20   0 /var/lib/snapd/dbus-1/system-services/

```

---
6. >Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.

Системный вызов - `uname`
```
uname({sysname="Linux", nodename="vagrant", ...}) = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0

```

Альтернативное местоположение: `/proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}`.
Информацию можно найти в справке `man 2 uname`:
```
NOTES

     This  is  a  system call, and the operating system presumably knows its name, release and version.  It also knows what hardware it runs on.  So, four of the fields of the struct are meaningful.  On the other
       hand, the field nodename is meaningless: it gives the name of the present machine in some undefined network, but typically machines are in more than one network and have several names.  Moreover, the  kernel
       has no way of knowing about such things, so it has to be told what to answer here.  The same holds for the additional domainname field.

       To this end, Linux uses the system calls sethostname(2) and setdomainname(2).  Note that there is no standard that says that the hostname set by sethostname(2) is the same string as the nodename field of the
       struct returned by uname() (indeed, some systems allow a 256-byte hostname and an 8-byte nodename), but this is true on Linux.  The same holds for setdomainname(2) and the domainname field.

       The length of the fields in the struct varies.  Some operating systems or libraries use a hardcoded 9 or 33 or 65 or 257.  Other systems use SYS_NMLN or _SYS_NMLN or UTSLEN or _UTSNAME_LENGTH.   Clearly,  it is a bad idea to use any of these constants; just use sizeof(...).  Often 257 is chosen in order to have room for an internet hostname.
       Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.

```

---
7. >Чем отличается последовательность команд через `;` и через `&&` в bash? Например
```
root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#
```
И `;`, и `&&` являются операторами для организации конвейерных команд. Сценарии использования:

command1 ; command2 - в этом случае `command2` выполнится после `command1`независимо от результата работы первой.

command1 && command2 - в этом случае `command2` будет выполняться только если `command1` выолнится успешно (вернет код завершения 0).

>Есть ли смысл использовать в bash &&, если применить set -e?

```
vagrant@vagrant:~$ help set
...
-e  Exit immediately if a command exits with a non-zero status.
...
```
Использование `set -e` в скрипте прерывает его работу, если exit code будет отличен от нуля, и дальнейший запуск команд после этого выполнен не будет. Работа `set -e` в данном случае чем-то похожа на работу оператора `&&`. Следовательно, использование `&&` избыточно при использовании `set -e`. 

---
8. >Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?

-e прервать выполнение команды немедленно, если команда завершается с ненулевым статусом
-u неустановленные параметры и переменные считаются за ошибки, с выводом в stderr текста ошибки и выполнит завершение неинтерактивного вызова
-x вывод команд и их аргументов по мере их выполнения 
-o pipefail возвращаемое значение конвейера как статус последней команды для выхода с ненулевым статусом. Или вернет ноль, если ни одна команда не вышла с ненулевым статусом, т.е. все команды завершились без ошибок.

Данный режим bash прервет выполнения команд при наличие ошибок, а также показывает более детальный вывод результатов выполнения команд, в частности - наличие ошибок.

---

9. >Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

Самые часто встречающиеся статусы процессов (через команду `ps aux`):
* `S` процессы, которые ожидают события для завершения, "спящие процессы". Такой процесс можно прервать
* `I` бездействующие потоки/процессы ядра

Дополнительные статусы:
* `<`	с высоким приоритетом
* `N`	с низким приоритетом
* `L`	процесс имеет заблокированные в памяти сектора для постоянного использования, например для процессов ввода-вывода
* `s`	процесс - лидер сессии
* `l`	многопоточный процесс
* `+` процесс которые запущены из терминала и который не дает возможности далее пользоваться этим терминалов, пока процесс запущен (переднеплановый процесс) 

```
```

---
