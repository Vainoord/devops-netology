## 3.2. Работа в терминале
---
1. >Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
```
vagrant@vagrant:~$ type -a cd
cd is a shell builtin
```
Это встроенная в Shell команда. Для удобства работы в терминальной сессии 
логичнее использовать встроенные команды для различных функций, например, 
для перемещения по директориям. Если бы команда cd была внешней, то ее 
работа была бы ограничена внешним собственных окружением.

---
2. >Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`?

Для того чтобы избежать использования pipe, можно воспользоваться опцией 
-c:
```
   General Output Control
       -c, --count
              Suppress normal output; instead print a count of matching 
lines for each input file.  With the -v,
              --invert-match option (see below), count non-matching lines.
```
```
vagrant@vagrant:~$ cat file
foo_bar
bar_bar
vagrant@vagrant:~$ grep foo_bar file | wc -l
1
vagrant@vagrant:~$ grep foo_bar file -c
1
```
---
3. >Какой процесс с PID 1 является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?

```
vagrant@vagrant:~$ lsof -p 1
COMMAND PID USER   FD      TYPE DEVICE SIZE/OFF NODE NAME
systemd   1 root  cwd   unknown                      /proc/1/cwd 
(readlink: Permission denied)
systemd   1 root  rtd   unknown                      /proc/1/root 
(readlink: Permission denied)
systemd   1 root  txt   unknown                      /proc/1/exe 
(readlink: Permission denied)
systemd   1 root NOFD                                /proc/1/fd (opendir: 
Permission denied)
```
Родительский процесс на виртуальной машине - system(1)
```
vagrant@vagrant:~$ pstree -p
systemd(1)─┬─ModemManager(701)─┬─{ModemManager}(706)
           │                   └─{ModemManager}(710)
           ├─VBoxService(854)─┬─{VBoxService}(855)
           │                  ├─{VBoxService}(856)
           │                  ├─{VBoxService}(857)
           │                  ├─{VBoxService}(858)
           │                  ├─{VBoxService}(859)
           │                  ├─{VBoxService}(860)
           │                  ├─{VBoxService}(861)
           │                  └─{VBoxService}(862)

```
---
4. >Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?

В терминале 0:
```
vagrant@vagrant:~$ who
vagrant  pts/0        2022-06-20 08:26 (10.0.2.2)
vagrant  pts/1        2022-06-20 10:36 (10.0.2.2)
vagrant@vagrant:~$ ls -l /dev/pts
total 0
crw--w---- 1 vagrant tty  136, 0 Jun 20 10:39 0
crw--w---- 1 vagrant tty  136, 1 Jun 20 10:36 1
c--------- 1 root    root   5, 2 Jun 20 08:23 ptmx

vagrant@vagrant:~$ ls -l /nonexistentdirectory 2>/dev/pts/1
vagrant@vagrant:~$ 
```

Результат, отображаемый в терминале 1:
```
Last login: Mon Jun 20 08:26:26 2022 from 10.0.2.2
vagrant@vagrant:~$ ls: cannot access '/nonexistentdirectory': No such file or directory
```
----
5. >Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
```
vagrant@vagrant:~$ cat test_out
cat: test_out: No such file or directory
vagrant@vagrant:~$ cat test_in
123
Hello World!
--endfile

vagrant@vagrant:~$ cat <test_in >test_out
vagrant@vagrant:~$ cat test_out 
123
Hello World!
--endfile

```
---
6. >Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?

TTY:\
![test](screenshots/Screenshot1.png)


PTS: 
```
vagrant@vagrant:~$ tty
/dev/pts/0
vagrant@vagrant:~$ Hello from /dev/tty1
cat /proc/cpuinfo | grep sse^C
vagrant@vagrant:~$ echo "hi /dev/tty1. It's a /dev/pts/0" > /dev/tty1
```
---
7. >Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?

Команда `bash 5>$1` создает файловый дескриптор 5 и перенаправляет его в stdout (`$1`)
```
vagrant@vagrant:~$ ls -l /proc/$$/fd/
total 0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 0 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 1 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 2 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 255 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 5 -> /dev/pts/0
```
Команда `echo netology > /proc/$$/fd/5` направит вывод "netology" в дескриптор 5, затем в 
stdout, т.к. в stdout был ранее перенаправлен дескриптор 5
```
vagrant@vagrant:~$ bash 5>$1
vagrant@vagrant:~$ ls -l /proc/$$/fd/
total 0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 0 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 1 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 2 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 255 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jun 20 12:53 5 -> /dev/pts/0
vagrant@vagrant:~$ echo netology > /proc/$$/fd/5
netology
```

---
8. >Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty?

*Создаем конструкцию с выводом stdout и stderr:
```
vagrant@vagrant:~$ (ls -l & cat -fff)
cat: invalid option -- 'f'
Try 'cat --help' for more information.
total 12
-rw-rw-r-- 1 vagrant vagrant 16 Jun 17 13:16 file
-rw-rw-r-- 1 vagrant vagrant 28 Jun 20 12:45 test_in
-rw-rw-r-- 1 vagrant vagrant 28 Jun 20 12:46 test_out
```
*Делаем подмену дескрипторов:
```
vagrant@vagrant:~$ (ls -l & cat -fff) 5>&1 1>&2 2>&5 | wc -l
total 12
-rw-rw-r-- 1 vagrant vagrant 16 Jun 17 13:16 file
-rw-rw-r-- 1 vagrant vagrant 28 Jun 20 12:45 test_in
-rw-rw-r-- 1 vagrant vagrant 28 Jun 20 12:46 test_out
2
```
В результате вывод stdout не потерян, а stderr передан на вход через pipe. Через `wc -l` мы 
видим две строки stderr в количественном значении (т.е. "2")

---
9. >Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?

Команда `cat /proc/$$/environ` выведет начальное состояние переменных окружения при запуске 
текущего 
($$) процесса, в данном случае это начальное состояние переменных при запуске shell.
```
/proc/[pid]/environ
              This file contains the initial environment that was set when the currently 
executing  program  was
              started  via  execve(2).   The entries are separated by null bytes ('\0'), and 
there may be a null
              byte at the end.  Thus, to print out the environment of process 1, you would 
do:

                  $ cat /proc/1/environ | tr '\000' '\n'
```
Аналогичный вывод можно получить при выполнении команды `env`:
```
vagrant@vagrant:~$ env
SHELL=/bin/bash
PWD=/home/vagrant
LOGNAME=vagrant
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/vagrant
LANG=C.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
SSH_CONNECTION=10.0.2.2 62455 10.0.2.15 22
LESSCLOSE=/usr/bin/lesspipe %s %s
XDG_SESSION_CLASS=user
TERM=xterm-256color
LESSOPEN=| /usr/bin/lesspipe %s
USER=vagrant
SHLVL=2
XDG_SESSION_ID=3
LC_CTYPE=C.UTF-8
XDG_RUNTIME_DIR=/run/user/1000
SSH_CLIENT=10.0.2.2 62455 22
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
SSH_TTY=/dev/pts/0
_=/usr/bin/env

```

---
10. >Используя man, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.

`/proc/[pid]/cmdline` - файл, который хранит командную строку для процесса (если я правильно понимаю - это полный путь до файла, которым был запущен процесс) 
```
/proc/[pid]/cmdline
              This  read-only file holds the complete command line for the process, unless 
the process is a zom‐
              bie.  In the latter case, there is nothing in this file: that is, a read on 
this file will  return
              0  characters.   The  command-line  arguments appear in this file as a set of 
strings separated by
              null bytes ('\0'), with a further null byte after the last string.
```
Пример:
```
vagrant@vagrant:~$ cat /proc/854/cmdline 
/usr/sbin/VBoxService--pidfile/var/run/vboxadd-service.sh
vagrant@vagrant:~$ cat /proc/854/cmdline | xargs -0
/usr/sbin/VBoxService --pidfile /var/run/vboxadd-service.sh
```
`/proc/[pid]/exe` - файл, который содержит ссылку на файл, другую ссылку или 
директорию. 
Другими словами - это символическая ссылка. Вызов данной ссылки фактически вызовет 
копию файла, 
обратится к ссылке внутри нее или вернет директорию.

```
/proc/[pid]/exe
              Under Linux 2.2 and later, this file is a symbolic link containing the actual 
pathname of the exe‐
              cuted  command.   This symbolic link can be dereferenced normally; attempting 
to open it will open
              the executable.  You can even type /proc/[pid]/exe to run another copy of the 
same executable that
              is  being run by process [pid].  If the pathname has been unlinked, the 
symbolic link will contain
              the string '(deleted)' appended to the original pathname.  In a multithreaded  
process,  the  con‐
              tents of this symbolic link are not available if the main thread has already 
terminated (typically
              by calling pthread_exit(3)).
```

---
11. >Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.
```
grep sse /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 
clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology 
nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe 
popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single pti fsgsbase 
avx2 invpcid rdseed clflushopt md_clear flush_l1d arch_capabilities
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 
clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology 
nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe 
popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single pti fsgsbase 
avx2 invpcid rdseed clflushopt md_clear flush_l1d arch_capabilities
```
В данном случае последняя версии SSE - sse4.2

---
12. >При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:
```
vagrant@netology1:~$ ssh localhost 'tty'
not a tty
```
>Почитайте, почему так происходит, и как изменить поведение.

Исходя из похожих ситуаций, описанных в stackoverflow, отвечу так:

Возможно, что при входе в оболочку localhost ожидает, что соединение будет инициироваться 
пользователем. В данном случае инициатор - процесс, поэтому ssh сессия не устанавливается.
С ключами  -t -l (установка принудительного псевдотерминала и указания пользователя) 
подключение устанавливается, но сразу же завершается.

```
vagrant@vagrant:~$ ssh -t -l vagrant localhost 'tty'
vagrant@localhost's password: 
/dev/pts/1
Connection to localhost closed.
```

---
13. >Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись reptyr. Например, так можно перенести в screen процесс, который вы запустили по ошибке в обычной SSH-сессии.

```
NAME
       reptyr - Reparent a running program to a new terminal
```
Попробовал пользоваться reptyr по этой инструкции: 
https://github.com/nelhage/reptyr#readme

* Start a long running process, e.g. top\
* Background the process with CTRL-Z\
* Resume the process in the background: bg
* Display your running background jobs with jobs -l, this should look like this:
```[1]+ 4711 Stopped (signal) top
(The -l in jobs -l makes sure you'll get the PID)
```
* Disown the jobs from the current parent with disown top. After that, jobs will not show the job any more, but ps -a will.
* Start your terminal multiplexer of choice, e.g. tmux
* Reattach to the backgrounded process: reptyr 4711
* Detach your terminal multiplexer (e.g. CTRL-A D) and close ssh
* Reconnect ssh, attach to your multiplexer (e.g. tmux attach), rejoice!

В результате процесс `top` не прерван, не висит в background, а работает в сессии мультиплексора. В данном случае вместо screen используется tmux. Для переключения на него используется команда `tmux attach`. Для обратного перехода используется сочетание `ctrl+b d`.
 
```
vagrant@vagrant:~$ bg
-bash: bg: current: no such job

vagrant@vagrant:~$ ps -a
    PID TTY          TIME CMD
   1558 pts/1    00:00:01 reptyr
   1617 pts/0    00:00:00 ps

vagrant@vagrant:~$ pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─VBoxService───8*[{VBoxService}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─agetty
        ├─atd
        ├─cron
        ├─dbus-daemon
        ├─irqbalance───{irqbalance}
        ├─multipathd───6*[{multipathd}]
        ├─networkd-dispat
        ├─polkitd───2*[{polkitd}]
        ├─rsyslogd───3*[{rsyslogd}]
        ├─snapd───10*[{snapd}]
        ├─sshd───sshd───sshd───bash───pstree
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-network
        ├─systemd-resolve
        ├─systemd-udevd
        ├─tmux: server─┬─bash
        │              └─bash───reptyr
        ├─top───top
        └─udisksd───4*[{udisksd}]
```
---
14. >`sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без 
`sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo 
string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от 
`sudo echo` команда с `sudo tee` будет работать.

С правами обычного пользователя нельзя создавать и изменять файлы в директории /root. 
Так и shell, который запущен под обычным пользователем, также не даст перенаправить вывод в 
файлы вышеупомянутой директории.
`tee` позволяет читать данные с стандартного ввода и записать эти данные в стандартный вывод 
и в указанные файл/файлы. В данном примере перенаправлять поток данных будет tee с правами 
суперпользователя - для перенаправления будет создан отдельный процесс в системе. Поэтому 
конструкция `echo string | sudo tee /root/new_file` сработает в заданных условиях.
```
vagrant@vagrant:~$ echo string | sudo tee /root/new_file
string
vagrant@vagrant:~$ sudo cat /root/new_file
string

```
---
