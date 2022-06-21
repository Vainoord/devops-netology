## 3.1. Работа в терминале, лекция 1
---
1. >Какие ресурсы выделены по-умолчанию для VM?

 RAM = 1 Gb\
 CPUS = 2\
 Video memory = 4 Mb\
 HDD = 64 Gb

---
2. >Как добавить оперативной памяти или ресурсов процессора виртуальной машине?

В файле Vagrantfile:

```
config.vm.provider "virtualbox" do |vb|
# Display the VirtualBox GUI when booting the machine
vb.gui = true
   
       # Customize the amount of memory on the VM:
       vb.memory = 2048
       vb.cpus = 2
     end
```
---
3. >Ознакомиться с разделами man bash, почитать о настройках самого bash:

>Какой переменной можно задать длину журнала history, и на какой строчке manual это описывается?
   
   Переменные HISTFILESIZE и HISTSIZE.
   HISTFILESIZE - максимальное количество строк, которое может быть записано в bash_history
   HISTSIZE - максимальное количество строкб которое выводится на экран при вызове команды history

  
2105 HISTORY\
When the -o history option to the set builtin is enabled, the shell provides access to the command history, 
the list of commands previously typed.  The value of the HISTSIZE variable is used as the number of commands to save  
in  a  history list.   The  text  of the last HISTSIZE commands (default 500) is saved.  The shell stores each 
command in the history list prior to parameter and variable expansion (see EXPANSION above) but after history expansion is 
performed, subject to the values of the shell variables HISTIGNORE and HISTCONTROL.

On startup, the history is initialized from the file named by the variable HISTFILE (default ~/.bash_history).  
The file named by the value of HISTFILE is truncated, if necessary, to contain no more than the number of lines 
specified by the value  of HISTFILESIZE.  If HISTFILESIZE is unset, or set to null, a non-numeric value, or a numeric 
value less than zero, the history file is not truncated.  When the history file is read, lines beginning with the 
history comment character followed immediately by a digit are interpreted as timestamps for the following history 
line.  These timestamps are optionally displayed depending on the value of the HISTTIMEFORMAT variable.  When a shell with history  
enabled  exits, the last $HISTSIZE lines are copied from the history list to $HISTFILE.  If the histappend shell 
option is enabled (see the description of shopt under SHELL BUILTIN COMMANDS below), the lines are appended to the 
history file, otherwise the history file is overwritten.  If HISTFILE is unset, or if the history file is unwritable, the history is not 
saved.  If the HISTTIMEFORMAT variable is set, time stamps are written to the history file, marked with the history 
comment character, so  they  may  be  preserved  across  shell sessions.  This uses the history comment character to distinguish 
timestamps from other history lines.  After saving the history, the history file is truncated to contain no more than 
HISTFILESIZE lines.  If HISTFILESIZE is unset, or set to null, a non-numeric value, or a numeric value less than zero, the 
history file is not truncated.

>Что делает директива ignoreboth в bash?

   Директива ignoreboth позволяет не добавлять команды в bash_history, если они начинаются с пробела или если она уже 
имеется в истории.

   Из файла .bashrc:

```
# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth
```

   HISTCONTROL\
A  colon-separated list of values controlling how commands are saved on the history list.  If the list 
of values includes ignorespace, lines which begin with a space character are not saved in the history list.  A value 
of ignoredups causes lines matching the previous history entry to not be saved.  A value of ignoreboth is shorthand for ignorespace and ignoredups.

---
4. >В каких сценариях использования применимы скобки {} и на какой строчке man bash это описано?

* Зарезервированные слова
128 RESERVED WORDS

* Групповые команды, которые выполняются в текущей оболочке	
186    Compound Commands
192        { list; }    

* Использование в функциях, условных операторах
286        function name [()] compound-command [redirection]

* Shell переменные
401    Shell Variables
432        BASH_LINENO
440        BASH_SOURCE

* Массивы
671    Arrays
${name[subscript]}

* Расширения для создания нескольких файлов или директорий\
```
714 EXPANSION
727    Brace Expansion
```
```
vagrant@vagrant:~$ mkdir /usr/local/src/bash/{old,new,dist,bugs}
```
----
5. >С учётом ответа на предыдущий вопрос, как создать однократным вызовом touch 100000 файлов? Получится ли аналогичным образом создать 300000? Если нет, то почему?

* Создание 100000 файлов возможно следующей командой:
```
vagrant@vagrant:~$ touch {1..100000}.file
```
* 300000 файлов создать не получается - слишком длинный список аргументов. Не получится создать уже 120000 файлов, но 110000 удалось.
```
vagrant@vagrant:~$ touch {1..300000}.txt
-bash: /usr/bin/touch: Argument list too long
```
---
6. >В man bash поищите по /\[\[. Что делает конструкция [[ -d /tmp ]]

[[ -d /tmp]] Возвращает статус 0 (так называемый Exit code), т.к. директория /tmp 
существует.

```
CONDITIONAL EXPRESSIONS
-d file
              True if file exists and is a directory.
[[ expression ]]

EXIT STATUS
       Shell builtin commands return a status of 0 (true) if successful, and non-zero  
(false)  if  an  error  occurs
       while  they  execute.   All builtins return an exit status of 2 to indicate incorrect 
usage, generally invalid
       options or missing arguments.
```
Проверка:
```
vagrant@vagrant:~$ [[ -d "/tmp" ]] && echo $?
0
```
Если директория не существует, то вернется 1:
```
vagrant@vagrant:~$ [[ -d "/test" ]]
vagrant@vagrant:~$ echo $?
1
```
---
7. >Основываясь на знаниях о просмотре текущих (например, PATH) и установке новых переменных; командах, которые мы рассматривали, добейтесь в выводе type -a bash в виртуальной машине наличия первым пунктом в списке
```
bash is /tmp/new_path_directory/bash\
bash is /usr/local/bin/bash\
bash is /bin/bash\
```
```
vagrant@vagrant:~$ mkdir /tmp/new_path_dir
vagrant@vagrant:~$ cp /bin/bash /tmp/new_path_dir
vagrant@vagrant:~$ PATH=/tmp/new_path_dir/:$PATH
vagrant@vagrant:~$ type -a bash
bash is /tmp/new_path_dir/bash
bash is /usr/bin/bash
bash is /bin/bash
```
---
8. >Чем отличается планирование команд с помощью batch и at?

at, batch, atq, atrm - queue, examine, or delete jobs for later execution
at      executes commands at a specified time.
batch   executes commands when system load levels permit; in other words, when the load average drops below 
1.5, or the value specified in the invocation of atd.

at - запускает команды в заданное время
batch - запускает команды когда позволяют уровни загрузки системы; другими словами, когда средняя нагрузка падает 
ниже 1.5 или значение, указанное при вызове atd.

batch не принимает параметры, в отличии от at.

---
