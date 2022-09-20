### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-03-yaml/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

# Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
Нужно найти и исправить все ошибки, которые допускает наш сервис

#### Ответ

```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : "7175"
            },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
    }
```
Пояснение:

0. 7175 - не IP. Но это скорее ошибка сбора данных.
1. "ip" : "7175" -  значение 7175 взято в кавычки
2. Добавлена запятая между элементами массива []
3. "ip" : "71.78.22.43" - добавлены кавычки

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import time
import socket
import datetime
import yaml
import json
import os


def write_data(FPath, DataHosts, lst):
    """Write hostname and IP address into config file"""
    # write data into JSON file
    with open(str(FPath) + "host_data" + ".json", 'w') as DataFile:
        JSONData = json.dumps(DataHosts, indent=4)
        DataFile.write(JSONData)
        DataFile.write('\n')
    # write data into YAML file
    with open(FPath + "host_data" + ".yaml", 'w') as DataFile:
        DataFile.write('---\n')
        YAMLData = yaml.dump(lst)
        DataFile.write(YAMLData)
        DataFile.write('...')

# dictionary of servers being monitored with initials IP addresses for JSON data
dict_servers = {
    'drive.google.com':'0.0.0.0',
    'mail.google.com':'0.0.0.0',
    'google.com':'0.0.0.0'
    }
# list of same servers for YAML data
list_servers = []
for el in dict_servers:
    list_servers.append({el: dict_servers[el]})

# get current directory path
folder_path = os.getcwd() + '/'
# Create once .json and .yaml files with default ip for each hosts
write_data(folder_path, dict_servers, list_servers)
# set script running constantly
while True:
    # print current date
    flag = False
    print(str(datetime.datetime.now().strftime("%Y/%m/%d %H:%M:%S"))+':')
    for host_name in dict_servers:
        # print server's name and old IP
        print(f'{host_name} - {dict_servers[host_name]}')
        time.sleep(1)
        # getting a ip from dns server by dns hostname
        try:
            host_ip = socket.gethostbyname(host_name)
        except:
            print(f'Host {host_name} not found')
        # print message if host IP has been changed
        if (host_ip != dict_servers[host_name]):
            print(f'[Warning] host_name IP mismatch: {dict_servers[host_name]}, {host_ip}')
            dict_servers[host_name] = host_ip
            time.sleep(2)
            # update source dictionary
            dict_servers.update({host_name:host_ip})
        #update source list
        for element in list_servers:
            for i in element:
                if (i == host_name):
                    element[host_name] = host_ip
                    flag = True
                    break
        # add data from dictionary and list into .json and .yaml files
        write_data(folder_path, dict_servers, list_servers)
    print('')
    time.sleep(5)
```

### Вывод скрипта при запуске при тестировании:
```
[vainoord@vnrd-mypc:homework]$ python3 ip_receive.py
2022/09/19 21:37:07:
drive.google.com - 0.0.0.0
[Warning] host_name IP mismatch: 0.0.0.0, 142.251.36.46
mail.google.com - 0.0.0.0
[Warning] host_name IP mismatch: 0.0.0.0, 142.251.36.37
google.com - 0.0.0.0
[Warning] host_name IP mismatch: 0.0.0.0, 142.250.179.174

2022/09/19 21:37:21:
drive.google.com - 142.251.36.46
mail.google.com - 142.251.36.37
google.com - 142.250.179.174

2022/09/19 21:37:29:
drive.google.com - 142.251.36.46
mail.google.com - 142.251.36.37
google.com - 142.250.179.174

```

### json-файл(ы), который(е) записал ваш скрипт:
```json
{
    "drive.google.com": "142.250.179.206",
    "mail.google.com": "142.251.36.5",
    "google.com": "142.251.39.110"
}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
---
- drive.google.com: 142.251.36.46
- mail.google.com: 142.251.36.37
- google.com: 142.250.179.174
...
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:
   * Принимать на вход имя файла
   * Проверять формат исходного файла. Если файл не json или yml - скрипт должен остановить свою работу
   * Распознавать какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны
   * Перекодировать данные из исходного формата во второй доступный (из JSON в YAML, из YAML в JSON)
   * При обнаружении ошибки в исходном файле - указать в стандартном выводе строку с ошибкой синтаксиса и её номер
   * Полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов

### Ваш скрипт:
```python

```

### Пример работы скрипта:
???
