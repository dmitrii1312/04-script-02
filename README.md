## Обязательная задача 1

Есть скрипт:
```
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

Вопросы:

| Вопрос | Ответ |
|------------|------------|
| Какое значение будет присвоено переменной c? | int и str нельзя складывать |
| Как получить для переменной c значение 12? | c = int(str(a)+b) |
| Как получить для переменной c значение 3? | c = a+int(b) |

## Обязательная задача 2

Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```
#!/usr/bin/env python3

import os
import sys

gitdir = '~/netology/sysadm-homeworks'

bash_command = ["cd {}".format(gitdir), "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
  if result.find('modified') != -1:
    prepare_result = gitdir+'/'+result.replace('\tmodified:   ', '')
    print(prepare_result)
```

### Вывод скрипта при тестировании

```
$ ./task2.py
~/netology/sysadm-homeworks/1.tst
~/netology/sysadm-homeworks/2.tst
~/netology/sysadm-homeworks/3.tst
```


## Обязательная задача 3

Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

```
#!/usr/bin/env python3

import os
import sys

gitdir='~/netology/sysadm-homeworks'
delim='/'

if len(sys.argv) >=2:
  directory = sys.argv[1]
  if os.path.isdir(directory):
    gitdir = directory
    if gitdir[-1] == '/':
      delim=''
  else:
    print("The path {} doesn't exists".format(directory))
    exit()

gitdir=os.path.expanduser(gitdir)
if not os.path.isdir(gitdir+delim+'.git'):
  print("The path {} is not a git repository".format(gitdir))
  exit()

bash_command = ["cd {}".format(gitdir), "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
  if result.find('modified') != -1:
    prepare_result = gitdir+delim+result.replace('\tmodified:   ', '')
    print(prepare_result)
```

### Вывод скрипта при тестировании

```
~$ ./task2.py /home/vagrant/netology/sysadm-homeworks
/home/vagrant/netology/sysadm-homeworks/1.tst
/home/vagrant/netology/sysadm-homeworks/2.tst
/home/vagrant/netology/sysadm-homeworks/3.tst
```
```
$ ./task2.py /home/vagrant/netology/
The path /home/vagrant/netology/ is not a git repository
```
```
$ ./task2.py /home/vagrant/netolo
The path /home/vagrant/netolo doesn't exists
```

## Обязательная задача 4
Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: drive.google.com, mail.google.com, google.com.

```
#!/usr/bin/env python3
import socket
import time

services = {'drive.google.com':'', 'mail.google.com':'', 'google.com':''}
for key, value in services.items():
  services[key]=socket.gethostbyname(key)

while True:
  for key, value in services.items():
    newip=socket.gethostbyname(key)
    if newip!=value:
      print("[ERROR] {} IP mismatch: {} {}".format(key,value,newip))
      services[key]=newip
    else:
      print("{} - {}".format(key,value))
  time.sleep(5)
```
### Вывод скрипта при тестировании

```
$ ./task4.py
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 64.233.165.101
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 64.233.165.101
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 64.233.165.101
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
[ERROR] google.com IP mismatch: 64.233.165.101 173.194.222.139
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
[ERROR] google.com IP mismatch: 173.194.222.139 173.194.222.113
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 173.194.222.113
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 173.194.222.113
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 173.194.222.113
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 173.194.222.113
drive.google.com - 173.194.73.194
mail.google.com - 173.194.221.19
google.com - 173.194.222.113
^CTraceback (most recent call last):
  File "./task4.py", line 19, in <module>
    time.sleep(5)
KeyboardInterrupt
```
