### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-02-py/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | Никакое, `a` и `b` имеют разный тип переменной - "TypeError: unsupported operand type(s) for +: 'int' and 'str'"  |
| Как получить для переменной `c` значение 12?  | Поменять тип переменной `a`: c = str(a) + b  |
| Как получить для переменной `c` значение 3?  | Поменять тип переменной `b`: c = a + int(b)  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

# поменял путь на свой для проверки
bash_command = ["cd ~/netology/devops-netology/", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
# переменная is_change не используется, убираем
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
# break прерывает работу цикла после первой отработки if, поэтому выводился только один modified файл
```

### Вывод скрипта при запуске при тестировании:
```
[vainoord@vnrd-mypc:~]$ python3 netology/homework/python.py
README.md
has_been_moved.txt
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3
import os
#import sys library
import sys

# path to repository
rep_path = sys.argv[1]
# checking if the script is being used correctly
if len(sys.argv) != 2:
    print('Usage: python3 script_name /path/to/git/repository')
    exit(0)
# is the directory exist?
if not (os.path.exists(sys.argv[1])):
    print('Directory ' + rep_path + ' does not exist')
    exit(0)
# redirect stderr to stdout for catching git errors
bash_command = ["cd " + rep_path, "git status 2>&1"]
result_os = os.popen(' && '.join(bash_command)).read()
for result in result_os.split('\n'):
    # if directory is not a git repository
    if result.find('fatal') != -1:
        print('Directory ' + rep_path + ' is not a git repository')
    # if we have modified files
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
```

### Вывод скрипта при запуске при тестировании:
```
[vainoord@vnrd-mypc:homework]$ python3 python.py
Usage: python3 script_name /path/to/git/repository

[vainoord@vnrd-mypc:homework]$ python3 python.py /tmp/
Directory /tmp/ is not a git repository

[vainoord@vnrd-mypc:homework]$ python3 python.py /Users/vainoord/netology/devops-netology/
README.md
has_been_moved.txt
```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import time
import socket
import datetime

# dictionary of servers being monitored with initials IP addresses
servers = {
    'drive.google.com':'0.0.0.0',
    'mail.google.com':'0.0.0.0',
    'google.com':'0.0.0.0'
    }
# set script running constantly
while True:
    # print current date
    print(str(datetime.datetime.now().strftime("%Y/%m/%d %H:%M:%S"))+':')
    for host_name in servers:
        # print server's name and old IP
        print(f'{host_name} - {servers[host_name]}')
        time.sleep(1)
        # getting a ip from dns server by dns hostname
        try:
            hostip = socket.gethostbyname(host_name)
        except:
            print(f'Host {host_name} not found')
        # print message if host IP has been changed
        if (hostip != servers[host_name]):
            print(f'[ERROR] host_name IP mismatch: {servers[host_name]}, {hostip}')
            servers[host_name] = hostip
            time.sleep(3)
    print('')
    time.sleep(5)
```

### Вывод скрипта при запуске при тестировании:
```
[vainoord@vnrd-mypc:homework]$ python3 receive_ip.py
2022/08/18 23:03:07:
drive.google.com - 0.0.0.0
[ERROR] host_name IP mismatch: 0.0.0.0, 142.251.36.46
mail.google.com - 0.0.0.0
[ERROR] host_name IP mismatch: 0.0.0.0, 142.250.179.133
google.com - 0.0.0.0
[ERROR] host_name IP mismatch: 0.0.0.0, 142.251.39.110

2022/08/18 23:03:24:
drive.google.com - 142.251.36.46
[ERROR] host_name IP mismatch: 142.251.36.46, 216.58.214.14
mail.google.com - 142.250.179.133
google.com - 142.251.39.110

2022/08/18 23:03:35:
drive.google.com - 216.58.214.14
mail.google.com - 142.250.179.133
google.com - 142.251.39.110

2022/08/18 23:03:43:
drive.google.com - 216.58.214.14
mail.google.com - 142.250.179.133
google.com - 142.251.39.110
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз переносить архив с нашими изменениями с сервера на наш локальный компьютер, формировать новую ветку, коммитить в неё изменения, создавать pull request (PR) и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. Мы хотим максимально автоматизировать всю цепочку действий. Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым). При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. С директорией локального репозитория можно делать всё, что угодно. Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. Важно получить конечный результат с созданным PR, в котором применяются наши изменения.

### Ваш скрипт:
```python
???
```

### Вывод скрипта при запуске при тестировании:
```
???
```
