---
description: >-
  Инструкция по установке стека из веб/прокси-сервера NGINX, wsgi-сервера uWSGI,
  базы данных PostgreSQL и фреймворка Django (python 3.8.1) на виртуальном
  сервере Beget VPS с Linux Debian 10 Buster.
---

# Создание и настройка Beget VPS с Debian

## Настройка доступа к серверу по SSH

### Скачивание и установка клиента SSH

Для доступа по SSH из-под операционной системы Windows используем утилиту PuTTY, которую можно бесплатно получить на официальном сайте разработчика по ссылке [https://www.putty.org/](https://www.putty.org/).

![&#x413;&#x43B;&#x430;&#x432;&#x43D;&#x430;&#x44F; &#x441;&#x442;&#x440;&#x430;&#x43D;&#x438;&#x446;&#x430; putty.org](../../.gitbook/assets/image%20%288%29.png)

На главной странице нажимаем на ссылку "here" \(выделена на изображении выше\). На следующей странице скачиваем установочный файл в соответствии с разрядностью операционной системы.

![&#x423;&#x441;&#x442;&#x430;&#x43D;&#x43E;&#x432;&#x43E;&#x447;&#x43D;&#x44B;&#x435; &#x444;&#x430;&#x439;&#x43B;&#x44B; &#x434;&#x43B;&#x44F; Widnows](../../.gitbook/assets/image%20%2818%29.png)

После скачивания запускаем установочный файл и производим стандартную установку. 

{% hint style="info" %}
Пользователи Linux могут производить доступ к удаленному серверу при помощи командной строки и команды вида ssh username@host-ip
{% endhint %}

После установки PuTTY на Windows в меню "Пуск" -&gt; "Все приложения" станут доступны утилиты PuTTY и PuTTYgen в общей папке с названием PuTTY.

### Доступ к серверу при помощи существующего ключа SSH

Для доступа к серверу используем ранее установленную утилиту PuTTY. В поле "Host Name \(or IP address\)" необходимо ввести IP-адрес сервера, к которому осуществляется подключение. В панели слева раскрыть подменю "SSH" нажав на плюсик слева от названия раздела. 

![&#x413;&#x43B;&#x430;&#x432;&#x43D;&#x430;&#x44F; &#x43F;&#x430;&#x43D;&#x435;&#x43B;&#x44C; &#x443;&#x442;&#x438;&#x43B;&#x438;&#x442;&#x44B; PuTTY](../../.gitbook/assets/image%20%2840%29.png)

Далее в панели слева выбираем подраздел "Auth". В соответствующей панели справа нажимаем кнопку "Browse..." и посредством проводника Windows указываем путь к файлу приватного ключа SSH.

![&#x423;&#x43A;&#x430;&#x437;&#x430;&#x43D;&#x438;&#x435; &#x43F;&#x443;&#x442;&#x438; &#x43A; &#x43A;&#x43B;&#x44E;&#x447;&#x443; SSH](../../.gitbook/assets/image%20%284%29.png)

После указания пути к SSH-ключу, возвращаемся в первый раздел "Session" и нажимаем кнопку "Open".

![&#x41E;&#x442;&#x43A;&#x440;&#x44B;&#x432;&#x430;&#x435;&#x43C; &#x43F;&#x43E;&#x434;&#x43A;&#x43B;&#x44E;&#x447;&#x435;&#x43D;&#x438;&#x435; &#x43F;&#x43E; SSH](../../.gitbook/assets/image%20%2829%29.png)

Открывается командная строка с приглашением к вводу имени пользователя. 

![&#x41A;&#x43E;&#x43C;&#x430;&#x43D;&#x434;&#x43D;&#x430;&#x44F; &#x441;&#x442;&#x440;&#x43E;&#x43A;&#x430; &#x443;&#x442;&#x438;&#x43B;&#x438;&#x442;&#x44B; PuTTY](../../.gitbook/assets/image%20%2830%29.png)

Теперь вводим соответствующее имя пользователя и при необходимости парольную фразу для ключа SSH. Таким образом мы получаем доступ к серверу по SSH через утилиту-клиент PuTTY на Windows. 

### Добавление нового пользователя с доступом по SSH

Создадим нового стандартного пользователя Debian и добавим для него SSH-ключ для возможности удаленного доступа к серверу. Необходимо подключиться к удаленному серверу при помощи утилиты PuTTY и root-пользователя \(как описано в предыдущем разделе\). Затем необходимо выполнить следующие команды в командной строке утилиты PuTTY:

```text
# создаем системного пользователя Debian
# <username> заменяем на любое имя пользователя, например, user
# в ходе диалога указываем пароль и учетные данные
# затем подтверждаем создание нового пользователя, нажав клавишу Y и Enter
sudo adduser <username>

# добавляем пользователя в группу sudo
sudo usermod -aG sudo <username>

# проверим наличие созданного пользователя
id <username>
# перезаходим за созданного пользователя
su - <username>
# повышаем привелегии пользователя до root (возможно, придется ввести пароль)
sudo su
# выходим из-под root
exit
# выходим из-под созданного пользователя
```

#### 

#### Создаем SSH-ключ для нового пользователя &lt;username&gt;

Для этого используем утилиту PuTTYgen, которая была установлена ранее в одном пакете вместе с утилитой PuTTY. Соответственно, открываем её на Windows через "Пуск" -&gt; "Все приложения" -&gt; "PuTTY" -&gt; "PuTTYgen" .

![&#x421;&#x43E;&#x437;&#x434;&#x430;&#x43D;&#x438;&#x435; &#x43A;&#x43B;&#x44E;&#x447;&#x430; &#x432; PuTTYgen](../../.gitbook/assets/image%20%2832%29.png)

Указываем длину ключа 4096 бит, как указано на изображении выше, и нажимаем кнопку "Generate". При генерации ключа необходимо будет передвигать курсор в пустой серой области окна. 

![&#x421;&#x433;&#x435;&#x43D;&#x435;&#x440;&#x438;&#x440;&#x43E;&#x432;&#x430;&#x43D;&#x43D;&#x44B;&#x439; &#x43A;&#x43B;&#x44E;&#x447; SSH](../../.gitbook/assets/image%20%2823%29.png)

После генерации нового ключа SSH в утилите PuTTYgen появится общая информация и возможность сохранить данный ключ. При необходимости для усиленной защиты можно указать парольную фразу \(какое-либо предложение или слово латиницей без пробелов\), ввести её необходимо два раза в поля "Key passphrase" и "Confirm passphrase". Затем можно сохранить публичный и приватный ключи нажатием кнопок "Save public key" и "Save private key", соответственно, и указания папки для сохранения посредством проводника \(данные ключи позволяют получить удаленный доступ к серверу, поэтому стоит их хранить отдельно от файлов с публичным доступом\). Программу **не закрываем** потому, как значение в верхнем поле Key \(начинается с ssh-rsa\) потребуется скопировать на сервер в файл authorized\_keys \(можно его скопировать пока что в отдельный текстовый файл и сохранить, а утилиту закрыть\). Необходимо скопировать значение вида \(обратите внимание на область прокрутки справа\):

```text
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAgEAzB9RmDOgAbAx3LdkL/WaM9lGa9se875l5/zPfhQiQrDrcPg7dGgl+wgi58nilpjjdzET+uhBWVPgO3rpztAJzbYCqR9rp3gWdacid/VnFLLa7Wh//aEsXPxNZLoGC05L0Fe7pjhFwHn3MewX2LsOh2WCF4+NLp7qmjzr3eT5hiLv3JRUJJFaJd5Lb3MPQyrNroZHRhPslTjOIn0eIxiWuPQlt0HLC14xJKSTCAS3kEuG92/mM0ZaltMWSw8By26x8aFKdVuo47VTrakD8z3pWKJSqEjEoHZ2UTreDFCnZdtnB5eUH6IV31wUSQLiox8ljUnlqTlnGxg7CECzpnGBf9b5BrEvfSvKG37HKKzMwKJ7cRf27pHm8W4K30vA+sJ7qbWZwNmjEulQ+5SFcPisFs3Mf1WQFXADyb7GDFSsVmtTGZIbcXgNiQfYdeLhpT8tDCiMhm5dQ0VfNp+N6IqJmCmv3qvFZaW57HnD0wVJJ9+Dl4b8R2OL2mKGJPIe8eFyXJx3nFJoGSWEAUSAg2xJq4U1D+fXm8rY9ZBFcQJzcODIEroGJ/FXlclegAnu0PdgoObsLrUjO/kAWtdLkwd7vJGCdkCMcRBMRsUzUYlW/rtSb2FPJaq0IYcIbYzriIWIg7q/Q0YtFYrUP+tGJFqzZxRwfov7yjtgDdnuXBt8H68= rsa-key-20200210
```

Значение должно начинаться на "ssh-rsa.." и оканчиваться на "..rsa-key-xxxxxxx", где xxxxxxx представляет собой некоторое число.

В итоге мы имеем публичный и приватный ключи для доступа к удаленному серверу через PuTTY \(конвертировать в ключи для OpenSSH можно также в данной утилите PuTTYgen во вкладке "Conversions"\) и публичный ключ для вставки в файл authorized\_key на сервере. 

#### Добавляем публичный ключ на удаленный сервер

Далее необходимо зайти на удаленный сервер посредством PuTTY и пользователя, который уже имеет доступ по SSH. Открываем утилиту PuTTY, вводим ip-адрес сервера, указываем путь к SSH-ключу пользователя, под которым хотим зайти, как было описано ранее. И нажимаем кнопку "Open". Открывается командная строка утилиты PuTTY с предложением ввести имя пользователя. Вводим имя пользователя, под которым хотим зайти, при необходимости также вводим парольную фразу и заходим на удаленный сервер. В итоге командная строка будет иметь вид как на изображении ниже \(не учитывая оформление\).

![&#x41A;&#x43E;&#x43C;&#x430;&#x43D;&#x434;&#x43D;&#x430;&#x44F; &#x441;&#x442;&#x440;&#x43E;&#x43A;&#x430; PuTTY &#x43F;&#x43E;&#x441;&#x43B;&#x435; &#x432;&#x445;&#x43E;&#x434;&#x430; &#x43F;&#x43E;&#x43B;&#x44C;&#x437;&#x43E;&#x432;&#x430;&#x442;&#x435;&#x43B;&#x44F;](../../.gitbook/assets/image.png)

Далее выполняем ряд команд, чтобы открыть файл authorized\_key на удаленном сервере:

```text
# перезаходим за нового пользователя (при необходимости вводим пароль)
su - <username>
# создаем директорию .ssh
sudo mkdir ~/.ssh
# открываем в редакторе nano файл authorized_keys 
sudo nano ~/.ssh/authorized_keys
```

![&#x420;&#x435;&#x434;&#x430;&#x43A;&#x442;&#x438;&#x440;&#x43E;&#x432;&#x430;&#x43D;&#x438;&#x435; &#x444;&#x430;&#x439;&#x43B;&#x430; authorized\_key &#x432; nano](../../.gitbook/assets/image%20%2827%29.png)

Переводим курсор на пустую строку ниже. На своем компьютере Windows копируем ранее созданный ключ из текстового файла или из области Key утилиты PuTTYgen. 

![&#x421;&#x43E;&#x445;&#x440;&#x430;&#x43D;&#x435;&#x43D;&#x43D;&#x44B;&#x439; SSH-&#x43A;&#x43B;&#x44E;&#x447;&#x432;  ](../../.gitbook/assets/image%20%2813%29.png)

Затем наводим курсор мыши на открытое окно командной строки PuTTY, делаем окно активным щелчком левой кнопкой мыши и, далее, копируем ключ правым щелчком мыши на области командной строки.

![](../../.gitbook/assets/image%20%2820%29.png)

Далее сохраняем изменения посредством нажатия комбинации клавиш "Ctr+X" и далее клавиши Y \(не забываем указать английскую раскладку клавиатуры\) и Enter. В итоге мы добавили новый SSH ключ на удаленный сервер. 

Устанавливаем требуемые права на директорию `/home/<username>/.ssh` посредством следующих команд:

```text
sudo chmod 700 ~/.ssh/
sudo chmod 600 ~/.ssh/authorized_keys
sudo chown -R <username>:<username> ~/.ssh/
```

Теперь можно закрыть утилиту PuTTY и затем попробовать зайти под новым пользователем с новым ключом SSH \(также открываем утилиту PuTTY, указываем ip-адрес сервера и указываем путь к новому SSH-ключу, далее нажимаем Open и вводим новое имя пользователя &lt;username&gt; в командную строку\) .

### Доступ к серверу строго по SSH

В целях безопасности необходимо исключить удаленный доступ от root-пользователя, а также доступ пользователей по паролям. В итоге оставить только доступ для обычных пользователей при помощи ключей SSH, а для выполнения операций из-под root использовать утилиту sudo на Linux Debian. 

{% hint style="danger" %}
Необходимо быть уверенным, что есть возможность доступа при помощи любого **системного** пользователя Debian \(не root-пользователя\) и **соответствующего SSH-ключа** для данного пользователя, иначе в итоге данных операций получить доступ на сервер будет невозможно!
{% endhint %}

Заходим на удаленный сервер при помощи утилиты PuTTY под системным пользователем \(не root\). Выполняем следующие команды:

```text
# открываем в редакторе nano конфигурацию ssh (вводим пароль пользователя)
sudo nano /etc/ssh/sshd_config

# в файле нужно установить (расскомментировать) следующие значения
    LoginGraceTime 30    # на вход в систему выделяется 30 секунд
    PermitRootLogin no    # запрет удаленного доступа из-под root
    MaxAuthTries 3    # максимальное количество неудачных попыток входа
    Protocol 2     # ограничение на использование более слабых протоколов для доступа
    PubkeyAuthentication yes    # аутентификация по публичному ключу SSH
    AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2    # использование списка ключей из authorized_keys
    HostbasedAuthentication no    
    PasswordAuthentication no    # аутентификация по паролям, также повторяется в самом конце файла (повторяемый параметр удалить)
    PermitEmptyPasswords no    # допустимость пустых паролей
    RhostsRSAAuthentication no
    UsePAM yes    # использование API Pluggable Authentication Modules для доступа внешних программ
    # в конце файла удалить повтор PasswordAuthentication yes

# затем перезапускаем службу ssh 
sudo service sshd restart
```

Теперь, сервер более защищен, и удаленный вход доступен только для системных пользователей по ключам SSH. Ограничено время на выполнение входа и максимальное количество неудачных попыток входа. Для того, чтобы заблокировать доступ отдельному пользователю необходимо удалить его SSH ключ из соответствующего файла `/home/<username>/.ssh/authorized_keys` \(редактирование данного файла было описано выше\) и затем, при необходимости, можно также удалить аккаунт данного пользователя следующей командой:

```text
sudo killall -u <username>    # завершить все процессы пользователя
userdel -r <username>    # удалить учетную запись пользователя и его домашний каталог
```

Для дополнительной защиты от брутфорса можно воспользоваться утилитой fail2ban. Дополнительно настроить порты и т.п. Более подробно про повышение безопасности сервера при доступе к нему по SSH можно прочитать, например, по ссылке [https://www.ibm.com/developerworks/ru/library/au-sshlocks/index.html](https://www.ibm.com/developerworks/ru/library/au-sshlocks/index.html)

## Первичная настройка сервера Debian

Дальнейшие действия производим непосредственно на удаленном сервере Debian при подключении по SSH через утилиту PuTTY. В самом начале добавим возможность получения пакетов из non-free репозиториев, для этого необходимо отредактировать файл sources.list. Однако на Beget VPS данный файл создается автоматически и может быть затем перезаписан, ввиду этого  необходимо изменить параметр`apt_preserve_sources_list: true` в файле `/etc/cloud/cloud.cfg`, для этого используем следующие команды:

```text
# открываем файл редактором nano для добавления параметра
sudo nano /etc/cloud/cloud.cfg
# добавим в конец файла параметр apt_preserve_sources_list: true в файле и сохраним изменения

# редактируем файл sources.list (добавляем contrib non-free для каждого репозитория)
sudo nano /etc/apt/sources.list
# сохраняем изменения: Ctrl+X затем Y и Enter
```

Можно привести файл `/etc/apt/sources.list`к следующему виду:

```bash
deb http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster main contrib non-free
deb http://security.debian.org/ buster/updates main contrib non-free
deb-src http://security.debian.org/ buster/updates main contrib non-free
deb http://deb.debian.org/debian buster-updates main contrib non-free
deb-src http://deb.debian.org/debian buster-updates main contrib non-free
deb http://deb.debian.org/debian buster-backports main contrib non-free
deb-src http://deb.debian.org/debian buster-backports main contrib non-free
```

Далее выполняем ряд команд для установки необходимого окружения:

```bash
# повышаем права до root и переходим в домашнюю директорию root-пользователя
sudo su
cd ~
# обновляем зависимости и пакеты
apt-get -y update && apt-get -y dist-upgrade
# устанавливаем необходимые пакеты одной командной
sudo apt-get -y install libtiff5-dev libjpeg62-turbo-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev gcc g++ libc-dev curl man libffi-dev libssl-dev libbz2-dev libncursesw5-dev libgdbm-dev liblzma-dev libsqlite3-dev tk-dev uuid-dev libreadline-dev build-essential libncurses5-dev libnss3-dev wget ufw nginx git 
# обновляем зависимости и пакеты
apt-get -y update && apt-get -y dist-upgrade

# настраиваем firewall утилитой ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh && ufw allow 22/tcp && ufw allow 80/tcp && ufw allow 8000/tcp && ufw allow 443/tcp

# добавим репозиторий с последней версией NodeJS - 12 на текущий момент
# для установки других версий заменяем значение 12.x на необходимое
sudo curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
# установим последнюю версию
sudo apt-get install -y nodejs
sudo apt-get update && sudo apt-get install yarn
# обновляем зависимости и пакеты
apt-get -y update && apt-get -y dist-upgrade
# проверяем версии nodejs и npm
nodejs -v && node -v && npm -v

# установим python 3.8.1
# скачиваем исходник XZ с официального сайта (можно взять ссылку на любую версию)
cd /opt
curl -O https://www.python.org/ftp/python/3.8.1/Python-3.8.1.tar.xz
# разархивируем исходники
tar -xf Python-3.8.1.tar.xz
# сконфигурируем и проверим исходники для сборки, создаем MAKEFILE
cd Python-3.8.1
./configure --enable-optimizations
# запустим сборку из исходников (нужно будет подождать окончания сборки ~15 мин)
make
# компилируем и устанавливаем
make altinstall 
# выставляем приоритет запуска версий (чем выше последняя цифра - выше приоритет)
update-alternatives --install /usr/bin/python python /usr/local/bin/python3.8 2
# обновляем зависимости
apt-get -y update && apt-get -y dist-upgrade
# устанавливаем дополнительные пакеты pip
python -m pip install --upgrade pip
python3.8 -m pip install virtualenv
python3.8 -m pip install virtualenvwrapper
sudo apt-get -y install python3-dev
# обновляем зависимости
apt-get -y update && apt-get -y dist-upgrade
# очистим старые записи после обновления
hash -d pip
# проверим версии python и pip
python -V && pip -V

# включаем FireWall
ufw enable

# просмотрим установленные правила firewall 
ufw status

# переходим в директорию по умолчанию
cd ~

# Подключаем сервер к GitHub
# генерируем SSH ключ
yes ~/.ssh/id_rsa | ssh-keygen -q -t rsa -b 4096 -C "admin@alistorya.com" -N '' > /dev/null
# добавляем ключи к пользователю
ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
# выводим публичный ключ, который нужно скопировать и добавить в аккаунт на GitHub (Settings -> SSH and GPG keys)
cat ~/.ssh/id_rsa.pub
# затем проверяем подключение к GitHub
sudo ssh -vT git@github.com

# Установим дополнительные пакеты certbot для SSL/TLS
sudo apt install -y certbot python3-certbot-nginx

# обновляем зависимости
sudo apt-get -y update && apt-get -y dist-upgrade

# выводим основную информацию
cat /etc/os-release    # информация о релизе ОС
arch    # архитектура ОС
nodejs -v
node -v
npm -v
nginx -v && git --version && python -V && pip -V
which python
which python3.8
```

## Установка Django 

Среда на production сервере будет представлена следующим образом:

![Django on Production Server](../../.gitbook/assets/1562449042114.png)

То есть запрос от клиента будет поступать на HTTP Reverse Proxy Web-Server NGINX, затем через Unix-сокет запрос передается на uWSGI и далее по протоколу WSGI запрос клиента поступает фреймворку Django \(то есть к роутеру и дальше направляется на соответствующую сущность\).

Создадим пользователя для работы с Django и соответствующим окружением:

```bash
# создаем системного пользователя <DJANGO_USER>, указываем любое имя пользователя, например, djangouser
# в ходе диалога указываем пароль, учетные данные оставим пустыми
# затем подтверждаем создание нового пользователя, нажав клавишу Y и Enter
sudo adduser djangouser

# добавляем пользователя в группу sudo и www-data
sudo usermod -aG sudo djangouser
sudo usermod -aG www-data djangouser

# проверим наличие созданного пользователя
id djangouser
```

Заходим под новым пользователем и настраиваем окружение:

```bash
# заходим за пользователя djangouser
su - djangouser
# создаем виртуальную среду Python при помощи оболочки virtualenvwrapper
# пропишем path для нового пользователя
which virtualenvwrapper.sh
source /usr/local/bin/virtualenvwrapper.sh
# создадим директорию projects
sudo mkdir ~/projects
# откроем файл .bashrc в редакторе nano
sudo nano ~/.bashrc
# добавим в конец файла .bashrc следующие записи
export PROJECT_HOME=$HOME/projects      # optional
source /usr/local/bin/virtualenvwrapper.sh
# перезайдем за пользователя djangouser
exit
su - djangouser

```

#### 

### Создаем виртуальное окружение и устанавливаем Django

Команда `django-admin startproject <PROJECT_NAME>` создаст одноименный каталог и пакет проекта в каталоге, в котором она была запущена. Параметр `<destination>` позволяет задать целевой каталог, если команда запускается в другом каталоге.

```bash
# создаем виртуальное окружение проекта
mkvirtualenv djangoenv
# переходим в каталог /home/djangouser/.virtualenvs/djangoenv
cdvirtualenv
# устанавливаем django
pip install django
# выйдем из виртуалньой среды и из-под root также установим django глобально
deactivate
exit
pip install django
# вернемся в виртуальное окружение
su - djangouser
workon djangoenv
cdvirtualenv
# запросим версию django
django-admin --version
# создаем проект Django
django-admin startproject djproject
# переходим в директорию проекта
cd djproject
# выполняем миграции (пока что SQLite)
python manage.py migrate
# добавляем SERVER_IP в ALLOWED_HOSTS 
sudo nano djproject/settings.py
# запускаем проект Django с указанием SERVER_IP
python manage.py runserver <SERVER_IP>:8000
# открываем в браузере <SERVER_IP>:8000 для проверки
# если всё запустилось, то можно остановить Django на сервере Ctrl+C
deactivate
exit    # [root]
```

### Устанавливаем и настраиваем связку uWSGI/NGINX

```bash
# переходим в каталог проекта
cd /home/djangouser/.virtualenvs/djangoenv/djproject
# устанавливаем глобально uwsgi из-под root-пользователя
pip install uwsgi	
# создадим тестовый файл	
sudo nano test.py		
# в файл test.py добавляем следующее
def application(env, start_response):
	start_response('200 OK', [('Content-Type','text/html')])
	return [b"Hello World"] 
	
# проверяем подключение uWSGI из-под root, проверяем в браузере доступ к <SERVER_IP>:8000 и затем выходим Ctrl+C
uwsgi --http :8000 --wsgi-file /home/djangouser/.virtualenvs/djangoenv/djproject/test.py

# запускаем созданный тестовый файл как веб-приложение через uWSGI из-под djangouser
su - djangouser
cd /home/djangouser/.virtualenvs/djangoenv/djproject/
uwsgi --http :8000 --module djproject.wsgi
```

Для работы со статикой и медиа создаем папки static и media \(если нет в папке проекта\) и заносим для проверки в media изображение media.jpg:

```bash
mkdir media
mkdir static
sudo chmod -R 755 media/
sudo chmod -R 755 static/
sudo chown -R djangouser:djangouser media/
sudo chown -R djangouser:djangouser static/
cd media 
wget  https://i.redd.it/jsnlg9kwvtxx.jpg
mv jsnlg9kwvtxx.jpg media.jpg
cd ..
uwsgi --http :8000 --module djproject.wsgi
# переходим по <server-ip>:8000/media/media.jpg и если видим картинку, то всё хорошо
# если возникла ошибка, то нам нужно добавить MEDIA_ROOT
sudo nano djproject/settings.py
# добавляем только для development
MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')
MEDIA_URL = '/media/'
# если возникает ошибка - открываем файл urls.py
sudo nano djproject/urls.py
# приводим его к след. виду
from django.contrib import admin
from django.urls import path
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
]  + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

# теперь повторно запускаем и проверяем, что картинка открылась
uwsgi --http :8000 --module djproject.wsgi
# переходим по <server-ip>/media/media.jpg и если видим картинку, то всё хорошо
# если проблемы с доступом к файлу, то изменим права
sudo chmod 700 media

# добавим процессинг медиа файлов в settings.py
sudo nano djproject/settings.py
# примерно так
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                # ... the rest of your context_processors goes here ...
                'django.template.context_processors.media',
            ],
         },
    },
]
# Добавим параметр (если он отсутствует) STATIC_ROOT в конец файла settings.py
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

# запускаем виртуальное окружение и переносим статику
workon djangoenv
cdvirtualenv
cd djproject
python manage.py collectstatic
deactivate
```

Создаем файл конфигурации для NGINX, соответственно выполняем команду `sudo nano djproject_nginx.conf` и добавляем следующую конфигурацию:

```bash
upstream djproject {
	server unix:///home/djangouser/.virtualenvs/djangoenv/djproject/djproject.sock;
}
server {
  listen 80;
  listen [::]:80;
	server_name     45.84.225.101;
	charset     utf-8;
	client_max_body_size 75M;
	location /media  {
		alias /home/djangouser/.virtualenvs/djangoenv/djproject/media;
	}
	location /static {
		alias /home/djangouser/.virtualenvs/djangoenv/djproject/static;
	}
	location / {
		uwsgi_pass  djproject;
		include     /etc/nginx/uwsgi_params;
	}
}
```

Создаем симлинк для активации конфига и перезапускаем nginx:

```bash
sudo ln -s /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_nginx.conf /etc/nginx/sites-enabled/
sudo /etc/init.d/nginx restart
```

Проверим работу веб-сервера:

```bash
uwsgi --socket djproject.sock --wsgi-file test.py --chmod-socket=666
uwsgi --socket djproject.sock --module djproject.wsgi --chmod-socket=666
```

Создаем конфигурационный файл для uwsgi `sudo nano djproject_uwsgi.ini` и заносим в него следующее:

```bash
[uwsgi]
project = djproject
venv = djangoenv
usr = djangouser

# Корневая папка проекта (полный путь)
chdir = /home/%(usr)/.virtualenvs/%(venv)/%(project)/
# Django wsgi файл
module = %(project).wsgi:application
# полный путь к виртуальному окружению
home = /home/%(usr)/.virtualenvs/%(venv)
wsgi-file = %(chdir)%(project)/wsgi.py
plugins = python
# общие настройки master
master = true
# максимальное количество процессов
processes = 5
# полный путь к файлу сокета
socket = %(chdir)%(project).sock
# права доступа к файлу сокета
chmod-socket = 666
# очищать окружение от служебных файлов uwsgi по завершению
vacuum = true
```

Пробуем запустить uwsgi через файл с конфигурацией:

```bash
exit 		# [root]
uwsgi --ini /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_uwsgi.ini --uid djangouser
```

При наличии ошибок стоит смотреть журнал в файл `/var/log/nginx/error.log`

### Запускаем приложение в режиме Emperor

Создаем папки для активации конфигов uwsgi из-под root-пользователя:

```bash
mkdir /etc/uwsgi/
mkdir /etc/uwsgi/apps-available/
mkdir /etc/uwsgi/apps-enabled/
```

Добавляем симлинк из нашего приложения `(djproject_uwsgi.ini)` в `/apps-enabled/` для активации данной конфигурации:

```bash
sudo ln -s /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_uwsgi.ini /etc/uwsgi/apps-enabled/
```

Для запуска теперь используем параметр `--emperor`, в который передаем путь к файлам конфигурации всех наших приложений:

```bash
uwsgi --emperor /etc/uwsgi/apps-enabled --uid djangouser
```

Все конфигурации NGINX/uWSGI, с которыми мы работаем, теперь нужно активировать симлинком в соответствующие директории:

NGINX -&gt; `/etc/nginx/sites-enabled/`   
uWSGI -&gt; `/etc/uwsgi/apps-enabled/`

Основные конфигурации для веб-приложения будут храниться в папке с проектом, а файл uwsgi-params в `/etc/nginx/uwsgi_params`

### Создаем службу для системного запуска

Установим системный демон для запуска, создаем файл и записываем туда:

```bash
sudo nano /etc/systemd/system/djproject.uwsgi.service
```

В данный файл добавляем следующее:

```bash
[Unit]
Description=uWSGI Emperor Service
After=syslog.target

[Service]
ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/apps-enabled --uid djangouser 
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
Restart=always
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

Теперь мы можем запускать uwsgi и соответствующие проекты в режиме Emperor как системную службу \(запуск производим от root\):

Запускаем и включаем uWSGI при загрузке:

```bash
systemctl enable djproject.uwsgi.service
systemctl start djproject.uwsgi.service
```

Запускаем и включаем Nginx при загрузке:

```bash
systemctl enable nginx
systemctl start nginx
```

Перезагрузка и проверка конфигурации:

```bash
nginx -t
sudo /etc/init.d/nginx restart
systemctl restart djproject.uwsgi.service
```

### Установка и подключение базы данных PostgreSQL

Устанавливаем дополнительные пакеты

```bash
sudo apt-get -y install libpq-dev postgresql postgresql-contrib

su - djangouser
workon djangoenv
cdvirtualenv

sudo pip install psycopg2
```

Заходим под стандартным пользователем postgres в СУБД и создаем базу данных и пользователя:

```bash
sudo su - postgres 
psql
# вводим запрос для создания базы данных и пользователя
CREATE DATABASE djprojectdb with encoding='UNICODE';
CREATE USER djprojectdbusr with password '******************';
GRANT ALL PRIVILEGES ON DATABASE djprojectdb TO djprojectdbusr;
# выходим из psql и закрываем сеанс пользователя postgres
\q
exit
```

Добавляем подключение к БД в настройки Django: Открываем `sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/djproject/settings.py` 

Ищем запись с DATABASES = ... и заносим/меняем на:

```bash
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djprojectdb',
        'USER': 'djprojectdbusr',
        'PASSWORD': '******************',
        'HOST': 'localhost',
        'PORT': '',
    }
}
# при необходимости устанавливаем русский язык и зону
LANGUAGE_CODE = 'ru-ru' 
TIME_ZONE = 'Europe/Moscow'
```

Делаем миграции в БД и создаем аккаунт администратора:

```bash
cd djproject
pip install psycopg2
python manage.py migrate
python manage.py createsuperuser
deactivate
exit    # [root]
```

Перезапускаем и проверяем установку

```bash
nginx -t
sudo /etc/init.d/nginx restart
systemctl restart djproject.uwsgi.service
```

### Выпуск SSL сертификата для домена

Запускаем Certbot с данными эл. почты и домена \(потребуется добавить к DNS домена запись TXT с указанным кодом acme, подождать 15 минут и отправить на одобрение запрос\):

```bash
certbot certonly --preferred-challenges=dns --agree-tos -m admin@alistorya.com -d *.alistorya.com -d alistorya.com --manual --server https://acme-v02.api.letsencrypt.org/directory
```

Выводим данные о сертификате и ключе \(необходимо получить путь к файлам\):

```bash
echo -e "\033[32m $(certbot certificates) \033[0m"
```

Полученные данные:

```bash
Certificate Name: alistorya.com
    Domains: *.alistorya.com alistorya.com
    Expiry Date: 2020-05-12 11:41:34+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/alistorya.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/alistorya.com/privkey.pem
```

Теперь необходимо правильно настроить конфигурацию nginx и перезапустить службы: Обновим зависимости и отредактируем файл конфигурации NGINX:

```bash
# обновляем зависимости
sudo apt-get -y update && apt-get -y dist-upgrade
sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_nginx.conf
```

Окончательный вид файла конфигурации NGINX следующий:

```bash
upstream djproject {
	server unix:///home/djangouser/.virtualenvs/djangoenv/djproject/djproject.sock;
}
server {
  listen 80;
  listen [::]:80;
	server_name lnovus.online www.lnovus.online;
	return 301 https://$server_name$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl on;
        ssl_certificate /etc/letsencrypt/live/lnovus.online/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/lnovus.online/privkey.pem;

        server_name www.lnovus.online;
        return 301 $scheme://lnovus.online;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        charset     utf-8;
        
        ssl on;
        ssl_certificate /etc/letsencrypt/live/lnovus.online/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/lnovus.online/privkey.pem;
	
	      server_name lnovus.online;
	      client_max_body_size 75M;
	
				# путь к favicon.ico
        location = /favicon.ico {
                alias /home/djangouser/.virtualenvs/djangoenv/djproject/favicon.ico;
        }

	      
		    location /static {
								autoindex on;
	      	      alias /home/djangouser/.virtualenvs/djangoenv/djproject/static/;
	      }
	
				location /media  {
								autoindex on;
	      	      alias /home/djangouser/.virtualenvs/djangoenv/djproject/media/;
	      }
	
	      location / {
	      	      uwsgi_pass  djproject;
	      	      include     /etc/nginx/uwsgi_params;
	      }
}
```

Перезагрузка и проверка конфигурации:

```bash
nginx -t
sudo /etc/init.d/nginx restart
systemctl restart djproject.uwsgi.service
# при необходимости добавим домен в ALLOWED_HOSTS
```

Закрываем 8000 порт при помощи ufw:

```bash
ufw deny 8000/tcp
```

Теперь сайт можно открыть только если указать домен. Соответственно остается доступ на портах 22 для ssh подключения и 443 для https \(с 80 порта происходит перенаправление на https\).

### Разделение Dev и Production конфигураций Django

Чтобы создать настройки для сервера разработки dev.py и для продакшена production.py мы разнесем файл settings.py. Создаем папку settings в директории с файлом settings.py, затем переносим файл settings.py в папку settings и переименовываем его в base.py \(наши базовые настройки Django\). 

Создаем в папке settings файл dev.py \(настройки на время разработки\) и файл production.py \(настройки для боевого сервера\).

```bash
# Заходим за пользователя djangouser
su - djangouser
# переходим в папку проекта где лежит файл settings.py
cd /home/djangouser/.virtualenvs/djangoenv/djproject/djproject/
# создаем директорию settings
mkdir settings
# переносим файл settings.py в директорию settings и переименовываем
mv settings.py settings/base.py
```

В директории settings создаем файл dev.py `sudo nano settings/dev.py` и заносим в него следующее:

```bash
# импортируем общие базовые настройки
from .base import *

# режим отладки при разработке будет включен
DEBUG = True
# доступ будет разрешен с любого хоста
ALLOWED_HOSTS = ['*']

# подключаем при разработке базу данных SQLite
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.2/howto/static-files/

STATIC_URL = "/static/"
STATICFILES_FINDERS = (
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
)

MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')
MEDIA_URL = '/media/'

# импортируем все локальные конфиги
try:
    from .local_settings import *
except ImportError:
    pass
```

Далее также создаем файл production.py `sudo nano settings/production.py` и заносим в него следующее:

```bash
# также импортируем базовые настройки
from .base import *

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.environ['SECRET_KEY_VALUE']
# отключаем режим отладки
DEBUG = False
# указываем только основные хосты 
ALLOWED_HOSTS = ['lnovus.online', 'www.lnovus.online']

# подключаем базу данных PostgreSQL на production
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djprojectdb',
        'USER': 'djprojectdbusr',
        'PASSWORD': '**********',
        'HOST': 'localhost',
        'PORT': '',
    }
}

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.2/howto/static-files/

STATIC_URL = "/static/"
STATIC_ROOT = os.path.join(BASE_DIR, "static/")

# также импортируем локальные конфиги
try:
    from .local_settings import *
except ImportError:
    pass
```

Редактируем файл base.py `sudo nano settings/base.py` до следующего вида:

```bash
import os

# мы спустились на одну директорию ниже, чем было раньше
# поэтому изменим BASE_DIR на правильный
PROJECT_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
BASE_DIR = os.path.dirname(PROJECT_DIR)


INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]

MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]

ROOT_URLCONF = "astra.urls"

# используем стандартный шаблонизатор
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [os.path.join(BASE_DIR, "templates")],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
                "django.template.context_processors.media",
            ]
        },
    },
]

WSGI_APPLICATION = "astra.wsgi.application"


# Password validation
# https://docs.djangoproject.com/en/2.2/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        "NAME": "django.contrib.auth.password_validation.UserAttributeSimilarityValidator"
    },
    {"NAME": "django.contrib.auth.password_validation.MinimumLengthValidator"},
    {"NAME": "django.contrib.auth.password_validation.CommonPasswordValidator"},
    {"NAME": "django.contrib.auth.password_validation.NumericPasswordValidator"},
]


# Internationalization
# https://docs.djangoproject.com/en/2.2/topics/i18n/

LANGUAGE_CODE = "ru-ru"
TIME_ZONE = "Europe/Moscow"
USE_I18N = True
USE_L10N = True
USE_TZ = True
```

Теперь изменим ссылки на файлы конфигурации в manage.py и в wsgi.py:

```bash
sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/manage.py
sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/djproject/wsgi.py
sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/djproject/asgi.py
```

Заменим строку `os.environ.setdefault("DJANGO_SETTINGS_MODULE", "djproject.settings")`   
на `os.environ.setdefault("DJANGO_SETTINGS_MODULE", "djproject.settings.production")`

Перезагрузка и проверка конфигурации:

```bash
nginx -t
sudo /etc/init.d/nginx restart
systemctl restart djproject.uwsgi.service
```

Теперь, пока мы не создадим приложение и не укажем правильное перенаправление при переходе в браузере будет указана ошибка 404 \(страница не найдена\). Для проверки работоспособности Django переходим в панель администрирования и пробуем зайти под созданными ранее учетными данными администратора `domain.com/admin`

### Создание простого приложения Django

Далее создадим простое приложения для проведения различных проверок.

```bash
# за djangouser заходим в директорию проекта под вирт. окр.
su - djangouser
cd /home/djangouser/.virtualenvs/djangoenv/djproject
workon djangoenv
# создаем приложение testapp (будет создан каркас приложения)
python manage.py startapp testapp
# добавим новое приложение в INSTALLED_APPS
sudo nano djproject/settings/base.py
# В итоге приводим INSTALLED_APPS к следующему виду
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'testapp.apps.TestappConfig',    # новое приложение
]
```

Редактируем представление нового приложения:

```bash
sudo nano testapp/views.py

# testapp/views.py
from django.http import HttpResponse


def homePageView(request):
    return HttpResponse("Server working, but we havn't interfaces")
```

Создаем файл с перенаправлениями \(роутер для приложения\):

```bash
sudo nano testapp/urls.py

# testapp/urls.py
from django.urls import path

from .views import homePageView

urlpatterns = [
    path('', homePageView, name='home')
]
```

Редактируем глобальный роутер проекта:

```bash
sudo nano djproject/urls.py

# djproject/urls.py
from django.contrib import admin
from django.urls import path, include # new

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('testapp.urls')), # new
]

```

### Установка SECRET\_KEY как переменной системного сервиса

Для того, чтобы не выдавать секретный ключ при удаленном хранении кода, его необходимо поместить в переменную системного фонового процесса, и делать вызов непосредственно от туда.

{% hint style="danger" %}
Если SECRET\_KEY содержит символ %, то его стоит экранировать повтором % \(получится так %%\). Одинарные кавычки также экранируют другие служебные символы bash.
{% endhint %}

```bash
# в settings/production.py пропишем вызов переменной окружения
SECRET_KEY = os.environ["SECRET_KEY_VALUE"]

# заходим под пользователем <DJANGO_USER> и устанавливаем для него системную переменную окружения
su - djangouser
export SECRET_KEY_VALUE='*******************************************'
# добавим его также в .bashrc
sudo nano ~/.bashrc
# в конец файла добавляем следующую строку
export SECRET_KEY_VALUE='*******************************************'
# обращаем внимание на одинарные кавычки (они экранируют служебные символы Bash)
# выходим за root
exit
# передаем переменную системному сервису
sudo nano /etc/systemd/system/djproject.uwsgi.service
# добавляем туда --env таким образом
ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/apps-enabled --uid <DJANGO_USER> --env SECRET_KEY_VALUE='<value>'
# пароль к базе данных мы храним в отдельном файле настроек production.py
# причем удаленный доступ к БД закрыт (порт закрыт)

# перезапускаем демона
systemctl daemon-reload
# перезапускаем веб-сервера
systemctl restart djproject.uwsgi.service && sudo /etc/init.d/nginx restart
```

Теперь наш секретный ключ и пароль к базе данных будут всегда оставаться на сервере, где им самое место. Главное файлы конфигурации и настроек сервера исключать из контроля версий и т.п.

## Служебные команды Django и окружения

```bash
# проверка production сервера на уязвимости
su - djangouser
workon djangoenv
cdvirtualenv
cd djproject
python manage.py check --deploy

# управление службами/сервисами
# перезагрузить системного демона Emperor (фоновый процесс)
systemctl daemon-reload
# перезагрузка uWSGI (также команды start/stop)
systemctl restart djproject.uwsgi.service 
# перезагрузка NGINX
sudo /etc/init.d/nginx restart

# управление и проверка конфигурации NGINX
nginx -t
sudo service nginx <start|stop|restart>
# перезапускаем веб-сервера с прокси
systemctl restart djproject.uwsgi.service && sudo /etc/init.d/nginx restart

# работа с Django
# заходим за djangouser
su - djangouser
# запускаем виртуальное окружение
workon djangoenv

# директория проекта Django
cd /home/djangouser/.virtualenvs/djangoenv/djproject/

# файлы конфигурации uWSGI
cd /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_uwsgi.ini

# файлы конфигурации NGINX
/etc/nginx/nginx.conf
cd /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_nginx.conf
/etc/nginx/sites-available/

# пути к сертификату SSL
/etc/letsencrypt/live/lnovus.online/fullchain.pem
/etc/letsencrypt/live/lnovus.online/privkey.pem

# база данных PostgreSQL
psql
```

## Оптимизация веб-сервера NGINX

```bash
# редактируем файл конфигурации NGINX
sudo nano /etc/nginx/nginx.conf
# устанавливаем следующие значения
worker_connections 4000;

##
# `gzip` Settings
##

gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_min_length 256;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;

# для смены worker_rlimit_nofile можно выполнить системную команду
ulimit -n 200000
# или изменить файл /etc/security/limits.conf
```

## Настройки безопасности Django для production

```bash
# параметры безопасности production экземпляра
# необходимо добавить в <project_dir>/settings/production.py
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_REFERRER_POLICY = 'same-origin'
```

## Контроль версий Git, сервис GitHub и среда разработки

Все файлы конфигурации \(nginx/uwsgi\) необходимо занести в .gitignore, также как и настройки production.py и файл unix-сокета. 

Берем из основной папки файл **manage.py** и папки со статикой и медиа **static/** **media/**, из папки с настройками проекта файлы **settings.py**, **urls.py** и **wsgi.py**. При создании приложений также берем основные файлы из них \(models.py, urls.py, views.py и подобные\).

```bash
# переходим в основную папку проекта и инициализируем репозиторий
cd /home/djangouser/.virtualenvs/djangoenv/djproject/
git init

# создаем .gitignore
sudo nano .gitignore 
```

Добавляем в .gitignore следующее:

```bash
### Django ###
*.log
*.pyc
db.sqlite3
djproject_nginx.conf
djproject_uwsgi.ini
djproject/settings/production.py
*.pyo
*~
media/**
**/__pycache__/*
*.py[cod]

# C extensions
*.so

# PyInstaller
#  Usually these files are written by a python script from a template
#  before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Translations
*.mo 
*.pot

# PyBuilder
target/

# Unit test / coverage reports
htmlcov/
.tox/
.coverage
.cache
nosetests.xml
coverage.xml

# Sphinx documentation
docs/_build/

# Thumbnails
._*
favicon.ico

# IDE
.idea/
```

Устанавливаем глобальные параметры Git:

```bash
# настроим глобальные параметры git
git config --global user.name "ProductionServer01"
git config --global user.email "admin@alistorya.com"
git config --global core.pager "less -r"
```

### Добавление в новый репозиторий

Если мы хотим добавить проект в чистый репозиторий, который только что создали, то выполняем следующее:

```bash
git add .

# проверим, какие файлы и папки мы теперь отслеживаем
git status

# при необходимости удаляем папки и файлы из индекса
# без удаления самого файла --cached
git rm --cached <filename>
git rm -r --cached <foldername>

# делаем инициализирующий коммит
git commit -m "Initial commit"

# подключаем удаленный репозиторий
git remote add origin <ssh-link>

# отправляем файлы в удаленный репозиторий
git push -f origin master
```

### Получаем данные из удаленного репозитория

Если в удаленном репозитории содержатся файлы проекта Django, с которыми мы работали ранее, то нам необходимо их получить и выполнить слияние с файлами на сервере. Для этого выполняем следующее:

```bash
# переходим в основную папку проекта и инициализируем репозиторий
cd /home/djangouser/.virtualenvs/djangoenv/djproject/
# подключаем удаленный репозиторий
git remote add origin <ssh-link>
# получаем все изменения
git fetch --all
# проводим сброс
git reset --hard origin/master
# если находимся на другой ветке
git reset --hard origin/<branch_name>
# применяем последние изменения
git pull origin master
```

Команда `git fetch` получает последние файлы с удаленного репозитория без замены и слияния локальных файлов, а затем мы сбрасываем ветку до той, которую получили из удаленного репозитория, причем опция --hard заменит все файлы в рабочей ветке, которые совпадут с файлами из удаленного репозитория \(подобно --force\). 

Далее можно выполнить соответствующие команды в окружении Django:

```bash
# вход за djangouser
su - djangouser
# активируем виртуальное окружение и переходим в директорию проекта
workon djangoenv
cdvirtualenv
cd djproject

# выполняем необходимые операции
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic
python manage.py createsuperuser

# перезапускаем все службы
sudo systemctl daemon-reload && sudo /etc/init.d/nginx restart && systemctl restart djproject.uwsgi.service
```

### Настройка контроля версий

Для контроля версий используем связку Git-GitHub и три ветки, помимо master, а именно:

* **dev** - в данной ветке ведем разработку \(пока работаем одни и приложение имеет небольшие размеры, а в будущем можно её поделить на ветки featrure, bugfix/hotfix и т.п.\). Данную ветку сливаем в master;
* **master** - основная ветка при разработке - если есть несколько разработчиков, то изменения они сливают в master, где далее проводят тесты и прочие операции, чтобы отправить код на ветку stage;
* **stage** - ветка для pre-production тестирования и отладки, которая выкатывается на промежуточный сервер и после выполнения всех необходимых операций сливается в production;
* **production** - ветка для выкладки на основной сервер, которому имеют доступ пользователи. 

```bash
# просмотреть данные git
git status    # состояние проекта
git log    # журналы проекта

# работа с ветками репозитория
git branch -a    # показать все ветки репозитория
git branch stage    # создать новую ветку с названием new_feature
git branch    # текущая ветка

git checkout new_feature    # перейти на созданную ветку
git checkout master    # перейти на основную ветку проекта
git merge new_feature    # произвести слияние ветки new_feature с основной

# стандартные операции
# добавим все файлы проекта в локальный репозиторий
sudo git add . 
# сделаем коммит в локальный репозиторий
sudo git commit -m 'message'
# запушим изменения в удаленный репозиторий
sudo git push -u origin master 
# получим обновления из удаленного репозитория
sudo git pull origin 
```

Теперь мы можем работать в ветке dev и при окончании сливать её в master, где можно проводить дополнительные операции, либо сливать туда параллельно другие ветки из разработки и тестирования для разрешения конфликтов, а затем её сливать в stage при помощи GitHub PullRequest, а после отладки на stage сервере и разрешения конфликтов можно выкатывать изменения на ветку production также при помощи GitHub PullRequest.

Также создадим отдельный репозиторий на GitHub под код проекта, чтобы в будущем его можно было без проблем передать. Используем также подключение SSH, но с другим названием для файла ключа:

```bash
# генерируем SSH ключ
yes ~/.ssh/git_id_rsa | ssh-keygen -q -t rsa -b 4096 -C "admin@alistorya.com" -N '' > /dev/null
# добавляем ключи к пользователю
ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/git_id_rsa
# выводим публичный ключ, который нужно скопировать и добавить в аккаунт на GitHub (Settings -> SSH and GPG keys)
cat ~/.ssh/git_id_rsa.pub
# затем проверяем подключение к GitHub
sudo ssh -vT git@github.com
```

После этого мы подключили еще один аккаунт на GitHub. На GitHub создаем пустой репозиторий и копируем ссылку для SSH. Переходим в папку проекта и подключаем второй репозиторий \(удаленный репозиторий именуем как main\):

```bash
cd /home/djangouser/.virtualenvs/djangoenv/djproject
git remote add main <link-to-new-repo>
# отправляем изменения в новый репозиторий
git push -u main master    # или production
```

### Автоматический pull на stage сервер с master

Установить [Go Lang Env](https://golang.org/doc/install) для сборки из исходников Gitomatic \([git-o-matic](https://github.com/muesli/gitomatic/blob/master/README.md)\). Получить пакеты и собрать данную утилиту.

```bash
# устанавливаем go
cd /opt 
wget https://dl.google.com/go/go1.13.7.linux-amd64.tar.gz
tar -xvf go1.13.7.linux-amd64.tar.gz
sudo mv go /usr/local
# установим переменные окружения Go для текущей сессии
export GOROOT=/usr/local/go
export GOPATH=$HOME/Projects/goprojects
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
# проверяем установку
go version
go env

# обновляем зависимости
sudo apt-get -y update && apt-get -y dist-upgrade

# устанавливаем gitomatic
git clone https://github.com/muesli/gitomatic.git
cd gitomatic
go build
```



