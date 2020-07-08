# MiniDevLab on Linux (local network dev host)

## Первоначальная настройка
### Настройка репозиториев на Debian 10 buster
```bash
# редактируем файл sources.list (добавляем contrib non-free для каждого репозитория)
sudo nano /etc/apt/sources.list

# Следует привести файл /etc/apt/sources.list к следующему виду
deb http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster main contrib non-free
deb http://security.debian.org/ buster/updates main contrib non-free
deb-src http://security.debian.org/ buster/updates main contrib non-free
deb http://deb.debian.org/debian buster-updates main contrib non-free
deb-src http://deb.debian.org/debian buster-updates main contrib non-free
deb http://deb.debian.org/debian buster-backports main contrib non-free
deb-src http://deb.debian.org/debian buster-backports main contrib non-free

# сохраняем изменения: Ctrl+X затем Y и Enter
# обновим зависимости
sudo apt update
```

### Установим базовые компоненты

```bash
# повышаем права до root и переходим в домашнюю директорию root-пользователя
sudo su
cd
# устанавливаем sudo (если отсутствует)
apt-get install sudo
visudo	# файл с настройками
# Создаем пользователя (указываем пароль и др. данные) и добавляем его в sudo
sudo adduser <username>
sudo usermod -aG sudo <username>

# обновляем зависимости и пакеты
apt-get -y update && apt-get -y dist-upgrade
# устанавливаем необходимые пакеты одной командной (\ перенос строки для удобства)
sudo apt-get -y install \
	libtiff5-dev \
	libjpeg62-turbo-dev \
	zlib1g-dev \
	libfreetype6-dev \
	liblcms2-dev \
	libwebp-dev \
	tcl8.6-dev \
	tk8.6-dev \
	libc-dev \
	libffi-dev \
	libssl-dev \
	libbz2-dev \
	libncursesw5-dev \
	libgdbm-dev \
	liblzma-dev \
	libsqlite3-dev \
	libreadline-dev \
	build-essential \
	libncurses5-dev \
	libnss3-dev \
	tk-dev \
	uuid-dev \
	gcc \
	g++ \
	man \
	curl \
	wget \
	nginx \
	ufw \
	git 

# nginx установлен стандартными методами для локального сервера
# обновляем зависимости и пакеты
apt update

# настраиваем firewall утилитой ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh && ufw allow 22/tcp && ufw allow 80/tcp && ufw allow 443/tcp
# при необходимости не забываем открыть другие порты

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

# установим python 3.8.3 - последняя версия на данный момент
# скачиваем исходник XZ с официального сайта (можно взять ссылку на любую версию)
cd /opt
curl -O https://www.python.org/ftp/python/3.8.3/Python-3.8.3.tar.xz
# разархивируем исходники
tar -xf Python-3.8.3.tar.xz
# сконфигурируем и проверим исходники для сборки, создаем MAKEFILE
cd Python-3.8.3
./configure --enable-optimizations
# запустим сборку из исходников (нужно будет подождать окончания сборки ~15 мин)
make
# устанавливаем в системные директории
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
# Установим дополнительные пакеты certbot для SSL/TLS
sudo apt install -y certbot python3-certbot-nginx

# обновляем зависимости
sudo apt-get -y update && apt-get -y dist-upgrade

# выводим основную информацию
cat /etc/os-release    # информация о релизе ОС
arch    # архитектура ОС
nodejs -v && node -v && npm -v
git --version && python -V && pip -V
which python
which python3.8

# удаляем временные файлы и подчищаем после установки
rm /opt/Python-3.8.3.tar.xz
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

### Доступ из локальной сети

```
# смотрим сетевые интерфейсы
ip a 
# редактируем настройки сети
sudo nano /etc/network/interfaces
# меняем параметр dhcp на static и дописываем нужные параметры сети
iface ens33 inet static
	address 192.168.1.15
	netmask 255.255.255.0
	network 192.168.1.0
	broadcast 192.168.1.1
	dns-nameservers 192.168.1.1
	# dns-nameservers 8.8.8.8 8.8.4.4

# перезапускаем сеть
systemctl restart networking
# проверяем настройки сети
ip addr show
# если интерфейс не поднялся, то выполняем
sudo ifup enp0s3
```

### Доступ по SSH

Если мы хотим получать доступ к серверу по SSH, то стоит отключить возможность входа от суперпользователя и доступ по паролю.
Изначально создаем пользователя и добавляем его в sudo

```
# создаем системного пользователя Debian
sudo adduser <username>
# добавляем пользователя в группу sudo
sudo usermod -aG sudo <username>

# проверим наличие созданного пользователя
id <username>
# перезаходим за созданного пользователя
su - <username>
# повышаем привелегии пользователя до root
sudo su
# выходим из-под root
exit
# выходим из-под созданного пользователя
```

В итоге мы создали нового системного пользователя в Debian <username>.

Создаем ssh-ключ для данного пользователя
```
# установка ssh-server
apt-get install openssh-server
# включение сервера ssh
systemctl enable ssh
systemctl start ssh
service ssh start	# stop - выключение
# файл с настройками ssh
nano /etc/ssh/sshd_config
# вход с другого компьютера -> ssh username@host/ip
# перезапуск ssh сервера
sudo systemctl restart ssh

# на клиентской машине, с которой будем подключаться, создаем ssh-ключ (указываем путь к ключу)
ssh-keygen -t rsa -b 4096 -C "mail@mail.com"

# на Windows переходим в папку C:/Windows/System32/OpenSSH, где расположен ssh-client
# при необходимости в Параметр -> Приложения -> Доп. компоненты установим ssh-клиента
# если при вызове ssh-agent происходит ошибка, то выполняем в PowerShell
Set-Service ssh-agent -StartupType Manual

# теперь созданный ключ надо установить на сервер
# можно использовать такую команду
cat ~/.ssh/id_rsa.pub | ssh <username>@host_id 'cat >> ~/.ssh/authorized_keys'
# либо при наличии утилиту
ssh-copy-id -i $HOME/.ssh/id_rsa.pub <username>@host_id
# либо самостоятельно зайти на сервер и создать файл authorized_keys
sudo nano ~/.ssh/authorized_keys
# затем в файл нужно скопировать данные открытого ключа id_rsa.pub 
# далее установим права
chmod 700 ~/.ssh/
chmod 600 ~/.ssh/authorized_keys
chown -R <username>:<username> ~/.ssh/
```

Далее пробуем зайти по созданному пользователю и ключу на сервер при помощи PuTTY - ключ нужно преобразовать в PuttyGen в ppk и затем заходить под созданным пользователем. 
После этого пробуем добавить ключ в системный клиент ssh.
В Windows идем в папку C:/Windows/System32/OpenSSH, а в Linux запускаем команду сразу:
```
# запускаем службу
start ssh-agent
# добавляем ключ системному пользователю на клиентской машине
ssh-add ~/.ssh/id_rsa
# далее пробуем зайти
ssh <username>@host_id
# либо с использованием PuTTY
```

После того, как мы убедились, что мы имеем доступ по ключу и учетным данным созданного пользователя отключаем вход по паролям и с аккаунта суперпользователя root:
```
# открываем в редакторе nano конфигурацию ssh (вводим пароль пользователя)
sudo nano /etc/ssh/sshd_config
# в файле нужно установить (расскомментировать) следующие значения
    LoginGraceTime 30    # на вход в систему выделяется 30 секунд
    PermitRootLogin no    # запрет удаленного доступа из-под root
    MaxAuthTries 3    # максимальное количество неудачных попыток входа
    Protocol 2     # ограничение на использование более слабых протоколов
    PubkeyAuthentication yes    # аутентификация по публичному ключу SSH
    # использование списка ключей из authorized_keys
    AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
    HostbasedAuthentication no    # доступ с хоста без пароля
    PasswordAuthentication no    # аутентификация по паролям
    PermitEmptyPasswords no    # допустимость пустых паролей
    UsePAM yes    # использование API Pluggable Authentication Modules
    # в конце файла удалить повтор PasswordAuthentication yes

# выходим из файла Ctrl+X с сохранением - Y и Enter последовательно

# затем перезапускаем службу ssh 
sudo service sshd restart
```

Теперь удаленный вход доступен только для системных пользователей по ключам SSH. Ограничено время на выполнение входа и максимальное количество неудачных попыток входа.

Для того, чтобы заблокировать доступ отдельному пользователю необходимо удалить его SSH ключ из соответствующего файла `/home/<username>/.ssh/authorized_keys` (редактирование данного файла было описано выше) и затем, при необходимости, можно также удалить аккаунт данного пользователя следующими командами:
```
sudo killall -u <username>    # завершить все процессы пользователя
userdel -r <username>    # удалить учетную запись пользователя
```


### Отключение графического интерфейса сервера

```
# переходим в консоль по Ctrl+Alt+F1
# определим имя графического менеджера
aptitude search '~i~Px-display-manager'
# перейдем в многопользовательский режим 
systemctl set-default multi-user.target
# управление менеджером графики
sudo service lightdm <stop | start | restart> 
# просмотр текущего режима
systemctl get-default
# включение графического интерфейса
systemctl set-default graphical.target
# можем также удалить графический пакет (крайне не рекомендуется)
# sudo aptitude -y remove xserver-xorg-core
```

### Установка PostgreSQL 12.3

```bash
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL
sudo apt-get install postgresql-12

# Login like postgres
su - postgres
# Use psql - check version
psql
SELECT version();

# exit
\q
exit

# Install additional packages
sudo apt-get install libpq-dev postgresql-contrib
```

## WikiJS installation

Create postgres user for wikijs:
```
su - postgres
psql
CREATE DATABASE wikijsdb with encoding='UNICODE';
CREATE USER wikijsdbuser with password 'naI@93*w';
GRANT ALL PRIVILEGES ON DATABASE wikijsdb TO wikijsdbuser;
\q
exit
```

Install wikiJS
```
# Download the latest version of Wiki.js
cd opt
wget https://github.com/Requarks/wiki/releases/download/2.4.107/wiki-js.tar.gz
# Extract the package to the final destination of your choice
mkdir /home/wiki
tar xzf wiki-js.tar.gz -C /home/wiki
cd /home/wiki
# Rename the sample config file to config.yml
mv config.sample.yml config.yml
```

Edit the config file and fill in your database and port settings
`nano config.yml`
Port:
`port: 3000`

Database
```
db:
  type: postgres
  host: localhost
  port: 5432
  user: wikijsdbuser
  pass: naI@93*w
  db: wikijsdb
```

Data folder on the end of file:

```
dataPath: /home/wiki/data
```

Offline Mode
```
offline: true
```

 HTTPS
 You need both the private key (key) and certificate (cert) in PEM format
 ```
 ssl:
  enabled: true
  port: 3443
  provider: custom

  format: pem
  key: path/to/key.pem
  cert: path/to/cert.pem
  passphrase: null
  dhparam: null
 ```
 
 It's also possible to use a PFX (pem) formatted certificate instead:
 ```
 ssl:
  enabled: true
  port: 3443
  provider: custom

  format: pfx
  pfx: path/to/cert.pfx
  passphrase: null
  dhparam: null
 ```

Let's Encrypt allows for free, automated and auto-renewing SSL certificates for your wiki
```
ssl:
  enabled: true
  port: 3443
  provider: letsencrypt

  domain: wiki.yourdomain.com
  subscriberEmail: admin@example.com
```

Once your HTTPS is up and working correctly, you can enable HTTP to HTTPS redirection under the Administration Area > SSL.

More configs on https://docs.requarks.io/install/config

Open port and run wikijs:
```
ufw allow 3000/tcp
node server
```

Check ports using net-stat:
```
sudo apt-get install net-tools
netstat -nlp | grep 5432
```

### Run WikiJS as service

```
# Create a new file named wiki.service inside directory /etc/systemd/system
nano /etc/systemd/system/wiki.service

# Paste the following contents (assuming your wiki is installed at /var/wiki)
[Unit]
Description=Wiki.js
After=network.target

[Service]
Type=simple
# Add dir to wikijs files
ExecStart=/usr/bin/node /home/wiki/server
Restart=always
# Consider creating a dedicated user for Wiki.js here:
User=nobody
WorkingDirectory=/home/wiki

[Install]
WantedBy=multi-user.target

# Reload systemd
systemctl daemon-reload
# Run the service
systemctl start wiki
# Enable the service on system boot
systemctl enable wiki
```

You can see the logs of the service using `journalctl -u wiki`

При необходимости, также выставить права 666 на package.json

## Установка Jira & Confluence

Создание базы данных в postgres:
```
su - postgres
psql
# создание базы данных под confluence и пользователя для atlassian продуктов
CREATE DATABASE confluencedb with encoding='UNICODE';
CREATE USER atlasianusr with password 'naI@93*w';
GRANT ALL PRIVILEGES ON DATABASE confluencedb TO atlasianusr;
# создание базы данных под jira
CREATE DATABASE jiradb with encoding='UNICODE';
GRANT ALL PRIVILEGES ON DATABASE jiradb TO atlasianusr;
# вывести список доступных баз данных
\l;
# установка пароля для postgres
psql -U postgres -c "ALTER USER postgres PASSWORD 'gJo610kkB$'"
# выход
\q
exit
```

Настраиваем порты
```
ufw allow 11312/tcp	# confluence main port
ufw allow 11314/tcp	# confluence control port
ufw allow 11313/tcp	# jira main port
ufw allow 11315/tcp	# jira control port
```

Вначале установим Jira, чтобы можно было цетрализованно работать с пользователями.

### Установка Jira 

```
# скачиваем установочный пакет
mkdir /opt/jira/ && cd /opt/jira/
sudo wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-8.10.0-x64.bin
# даем права на исполнение и устанавливаем
sudo chmod a+x atlassian-jira-software-8.10.0-x64.bin
sudo ./atlassian-jira-software-8.10.0-x64.bin
# в ходе установки указываем директории /home/atlassian/jira и /home/atlassian/application-data/jira, порты 11313 и 11315 и соглашаемся на установку системного демона (y)
# команды для управления
sudo service jira <start | stop | restart>
systemctl  <start | stop | status | restart> jira.service
# базовые директории для jira
/home/atlassian/jira
/home/atlassian/application-data/jira
# конфиг сервера
/home/atlassian/jira/conf/server.xmly
```

Далее переходим к настройке jira в браузере по соответствующим IP-адресу и порту, где указываем данные для подключения к БД и данные нового пользователя. 

### Установка Confluence

```
# скачиваем установочный пакет
mkdir /opt/confluence/ && cd /opt/confluence/
sudo wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-7.6.0-x64.bin
# даем права на исполнение и устанавливаем
sudo chmod a+x atlassian-confluence-7.6.0-x64.bin
sudo ./atlassian-confluence-7.6.0-x64.bin
# в ходе установки указываем директории /home/atlassian/confluence и /home/atlassian/application-data/confluence, порты 11312 и 11314 и соглашаемся на установку системного демона (y)

# команды для управления
sudo service confluence <start | stop | restart>
systemctl  <start | stop | status | restart> confluence.service
# базовые директории для Confluence
/home/atlassian/confluence 
/home/atlassian/application-data/confluence
# конфиг сервера
/home/atlassian/confluence/conf/server.xml

# для полного удаления выполнить следующее
systemctl  stop confluence.service
cd /home/atlassian/confluence/
sudo chmod a+x uninstall
./uninstall
```

Далее переходим к настройке Confluence в браузере по соответствующим IP-адресу и порту, где указываем данные для подключения к БД и данные нового пользователя. 

Стоит также установить плагин Draw.io для создания диаграмм в Confluence. В управлении приложениями можно включить стандартные макросы HTML (по умолчанию они отключены). Не забываем их отключить при установке на production сервер.

## Установка GitLab

```
# установить требуемые зависимости
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates postfix

# добавить репозиторий
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
# установить gitlab-ce
sudo apt-get install gitlab-ce
# редактируем конфиг
sudo nano /etc/gitlab/gitlab.rb
# указываем в следующей строке доменное имя или ip-адрес
external_url 'http://example.com'
# можно изменить также следующие строки
nginx['enable'] = false # выключение встроенного nginx
web_server['external_users'] = ['www-data'] # пользователь, от имени которого работает веб-сервер
gitlab_rails['trusted_proxies'] = [ '127.0.0.1', '<внешний IP адрес сервера>' ] # параметр для определения IP адресов проксируемого сервера
gitlab_workhorse['listen_network'] = "unix" # протокол для работы с веб-сервером
gitlab_workhorse['listen_addr'] = "/var/opt/gitlab/gitlab-workhorse/socket" # интерфейс и порт для прослушивания
# сохраняем конфигурацию и применяем изменения
gitlab-ctl reconfigure
# в Debian 10 может потребоваться создать симлинк и затем повторно применить изменения
ln -s /usr/sbin/sysctl /bin/sysctl 
```

GitLab использует собственный инстанс PostgreSQL, но мы подключим его к общему. Для этого создадим базу данных и пользователя:
```
su - postgres
psql
CREATE DATABASE gitlabhq_production with encoding='UNICODE';
CREATE USER gitlabusr with password 'bFET9BD7!Wwc';
GRANT ALL PRIVILEGES ON DATABASE gitlabhq_production TO gitlabusr;
# дадим права суперпользователя для автоматической установки требуемых расширений
ALTER USER gitlabusr WITH SUPERUSER;
```

Затем нужно добавить в /etc/gitlab/gitlab.rb следующее:
```
# откроем в редакторе
sudo nano /etc/gitlab/gitlab.rb

# добавим настройки базы данных
postgresql['listen_address'] = '0.0.0.0'
postgresql['port'] = 5432
postgresql['md5_auth_cidr_addresses'] = %w()
postgresql['trust_auth_cidr_addresses'] = %w(127.0.0.1/24)
postgresql['sql_user'] = "gitlabusr"

##! SQL_USER_PASSWORD_HASH can be generated using the command `gitlab-ctl pg-password-md5 gitlab`,
##! where `gitlab` is the name of the SQL user that connects to GitLab.
postgresql['sql_user_password'] = "SQL_USER_PASSWORD_HASH"

# force ssl on all connections defined in trust_auth_cidr_addresses and md5_auth_cidr_addresses
postgresql['hostssl'] = false

gitlab_rails['db_host'] = '127.0.0.1'
gitlab_rails['db_port'] = 5432
gitlab_rails['db_username'] = "gitlabusr"
gitlab_rails['db_password'] = "bFET9BD7!Wwc"
```

Добавим Redis:
```
# установим redis-server
sudo apt-get install redis-server
# перейдем в консоль redis
redis-cli
ping	# output -> PONG
# установка прошла успешно
```

Внесем изменения в конфиг GitLab:
```
# откроем в редакторе
sudo nano /etc/gitlab/gitlab.rb

# добавим настройки redis
redis['enable'] = false
# Redis via TCP
gitlab_rails['redis_host'] = '127.0.0.1'
gitlab_rails['redis_port'] = 6379
# Password to Authenticate to alternate local Redis if required
gitlab_rails['redis_password'] = 'Redis Password'
```

Настроим Redis:
```
# откроем файл конфигурации
sudo nano /etc/redis/redis.conf
# находим и расскомментим строку
bind 127.0.0.1 ::1
# установим пароль
# ищем строку requirepass foobared - расскоментим и изменим foobared на пароль
requirepass bxP5xk%1LRx3

# сохраняем и перезапускаем 
sudo systemctl restart redis.service
```

Далее переконфигурируем gitlab и перезапустим postgres:
```
sudo gitlab-ctl reconfigure
gitlab-ctl restart postgresql

# посмотреть лог
sudo nano /var/log/gitlab/gitlab-rails/production.log
```

При первой загрузке страницы GitLab будет предложено ввести новый пароль для пользователя root, при помощи которого можно получить доступ к Admin Area. Если есть проблемы с доступом через localhost, то стоит его добавить в Admin Area -> Settings -> Network -> Outbound requests -> установить галочку Allow requests to the local network from web hooks and services и в поле ниже ввести хост сервера (127.0.0.1 и т.п.), можно также указывать конкретный порт.


# Install Seafile pro 7.1.5

```bash
# Get Seafile Pro 7.1.5 package and put in `/home` directory

# -------------------------------------------
# Additional requirements
# -------------------------------------------
apt-get update

# On Ubuntu 18.04
apt-get install -y python3 python3-setuptools python3-pip python3-ldap memcached openjdk-8-jre libmemcached-dev libreoffice-script-provider-python libreoffice pwgen curl nginx

pip3 install --timeout=3600 Pillow pylibmc captcha jinja2 sqlalchemy psd-tools django-pylibmc django-simple-captcha

service memcached start

# On Debian 10 Buster
apt-get install -y python3 python3-setuptools python3-pip python3-ldap memcached libmemcached-dev libreoffice-script-provider-python libreoffice pwgen curl nginx 
apt-get install -y nvidia-openjdk-8-jre
apt-get install -y zlib1g-dev libssl-dev python3-dev build-essential
sudo apt-get install ttf-wqy-microhei ttf-wqy-zenhei xfonts-wqy
sudo apt-get install poppler-utils

pip3 install Pillow --upgrade 
pip3 install --timeout=3600 Pillow pylibmc captcha jinja2 sqlalchemy psd-tools django-pylibmc django-simple-captcha

service memcached start

# -------------------------------------------
# Setup Nginx
# -------------------------------------------

rm /etc/nginx/sites-enabled/*
nano /etc/nginx/sites-available/seafile_nginx.conf
ln -sf /etc/nginx/sites-available/seafile_nginx.conf /etc/nginx/sites-enabled/seafile_nginx.conf
service nginx restart

# -------------------------------------------
# MariaDB
# -------------------------------------------

apt-get install -y mariadb-server
service mysql start

mysqladmin -u root password g6O9DVxi8$n6 	# <root-password>

# теперь настроим безопасное подключение к БД
sudo mysql_secure_installation 	# только на смену пароля отвечаем No

# seafile во время установки запросит пароль от root и установит требуемые БД

# проверяем установку базы данных
sudo service mysql status	# sudo service mysql start|restart|stop

# подключение можно выполнять так
mysql -u <имя_пользователя> -p	

# -------------------------------------------
# Seafile
# -------------------------------------------

mkdir -p /home/seafile
mkdir -p /home/seafile/installed
cd /home/seafile
mv /home/seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz /home/seafile/seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz
tar xzf seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz
mv /home/seafile/seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz /home/seafile/installed/seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz

# C отключенным шелом и одноименной группой
# useradd seafile -b /home/ -m -U -s /bin/false
useradd --system --comment "seafile" seafile --home-dir  /home/seafile -m -U -s /bin/false
# задаем пароль
passwd seafile

# Предоставим Nginx, доступ в домашнюю директорию пользователя seafile
usermod -a -G seafile www-data 	# Добавить пользователя www-data в группу seafile

# gpasswd -d www-data seafile - Удалить пользователя www-data из группы seafile
# id www-data - Проверить в какие группы входит пользователь www-data

cd /home/seafile/seafile-pro-server-7.1.5

# -------------------------------------------
# Create ccnet, seafile, seahub conf using setup script
# -------------------------------------------

# -w <mysql-seafile-password> -r <mysql-root-password>
# more on https://seafile.gitbook.io/seafile-server-manual/deploying-seafile-under-linux/deploying-seafile-with-mysql

./setup-seafile-mysql.sh auto -u seafile -w Cj6yaRO#TWv6 -r g6O9DVxi8$n6 -p 17100

# -------------------------------------------
# Configure Seafile WebDAV Server(SeafDAV)
# -------------------------------------------

# -------------------------------------------
# Configuring seahub_settings.py
# -------------------------------------------

python3 /home/seafile/seafile-pro-server-7.1.5/pro/pro.py setup --mysql --mysql_host=127.0.0.1 --mysql_port=3306 --mysql_user=seafile --mysql_password=Cj6yaRO#TWv6 --mysql_db=seahub_db

# -------------------------------------------
# Fix permissions
# -------------------------------------------

chown seafile:seafile -R /home/seafile

# -------------------------------------------
# Start seafile server
# -------------------------------------------

sudo -u seafile /home/seafile/seafile-pro-server-7.1.5/seafile.sh start
sudo -u seafile /home/seafile/seafile-pro-server-7.1.5/seahub.sh start 17101

# wait and stop

sudo -u seafile /home/seafile/seafile-pro-server-7.1.5/seafile.sh stop
sudo -u seafile /home/seafile/seafile-pro-server-7.1.5/seahub.sh stop

# -------------------------------------------
# Fix other permissions
# -------------------------------------------

chown seafile:seafile -R /tmp/seafile-office-output/

# -------------------------------------------
# Change seafdav.conf
# -------------------------------------------

cd ../conf
nano seafdav.conf

[WEBDAV]
enabled = true
port = 17200	# default 8080
fastcgi = false
share_name = /seafdav
```

# Seafile NGINX config

Add to `/etc/nginx/sites-available/seafile_nginx.conf`

```NGINX
log_format seafileformat '$http_x_forwarded_for $remote_addr [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $upstream_response_time';

server {
    listen 80;
    server_name seafile.example.com;	# add server ip or domain here

    proxy_set_header X-Forwarded-For $remote_addr;

    location / {
         proxy_pass         http://127.0.0.1:17101;
         proxy_set_header   Host $host;
         proxy_set_header   X-Real-IP $remote_addr;
         proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header   X-Forwarded-Host $server_name;
         proxy_set_header   X-Forwarded-Proto $scheme;
         proxy_read_timeout  1200s;

         # used for view/edit office file via Office Online Server
         client_max_body_size 0;

         access_log      /var/log/nginx/seahub.access.log seafileformat;
         error_log       /var/log/nginx/seahub.error.log;
    }
    
    location /seafhttp {
         rewrite ^/seafhttp(.*)$ $1 break;
         proxy_pass http://127.0.0.1:17100;
         client_max_body_size 0;
         proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_connect_timeout  36000s;
         proxy_read_timeout  36000s;

        access_log      /var/log/nginx/seafhttp.access.log seafileformat;
        error_log       /var/log/nginx/seafhttp.error.log;
    }
    location /media {
        root /home/seafile/seafile-server-latest/seahub;
    }
    location /seafdav {
        proxy_pass         http://127.0.0.1:17200/seafdav;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_read_timeout  1200s;

        client_max_body_size 0;

        access_log      /var/log/nginx/seafdav.access.log seafileformat;
        error_log       /var/log/nginx/seafdav.error.log;
    }
}
```

# Seafile configs [/opt/seafile/conf]

## ccnet.conf

```ini
[General]
SERVICE_URL = http://127.0.0.1:17101

[Database]
ENGINE = mysql
HOST = 127.0.0.1
PORT = 3306
USER = seafile
PASSWD = Cj6yaRO#TWv6
DB = ccnet_db
CONNECTION_CHARSET = utf8
```

## gunicorn.conf.py

```python
import os

daemon = True
workers = 5

# default localhost:8000
bind = "127.0.0.1:17101"

# Pid
pids_dir = '/home/seafile/pids'
pidfile = os.path.join(pids_dir, 'seahub.pid')

# for file upload, we need a longer timeout value (default is only 30s, too short)
timeout = 1200

limit_request_line = 8190
```


## seafevents.conf

```ini
[DATABASE]
type = mysql
host = 127.0.0.1
port = 3306
username = seafile
password = Cj6yaRO#TWv6
name = seahub_db

[AUDIT]
enabled = true

[INDEX FILES]
enabled = true
interval = 10m

highlight = fvh

## If true, indexes the contents of office/pdf files while updating search index
## Note: If you change this option from "false" to "true", then you need to clear the search index and update the index again. See the FAQ for details.
index_office_pdf = true

[OFFICE CONVERTER]
enabled = true
workers = 1
## the max size of documents to allow to be previewed online, in MB. Default is 2 MB
## Preview a large file (for example >30M) online will freeze the browser.
max-size = 2

[SEAHUB EMAIL]
enabled = true

## interval of sending Seahub email. Can be s(seconds), m(minutes), h(hours), d(days)
interval = 30m

# Enable statistics
[STATISTICS]
enabled=true
```

## seafile.conf

```ini
[fileserver]
port = 17100

[database]
type = mysql
host = 127.0.0.1
port = 3306
user = seafile
password = Cj6yaRO#TWv6
db_name = seafile_db
connection_charset = utf8

[quota]
# Размер дисковой квоты для всех пользователей = Гб;
default = 10

[history]
# Количество дней хранения истории изменений файлов;
keep_days = 15

[library_trash]
# Количество дней, после которых произойдет авто очистка корзины;
expire_days = 30
```

## seahub_settings.py

```python
# -*- coding: utf-8 -*-
SECRET_KEY = "b'k)4zzv=9%#)7$=hqz%dp%85u=ht1z$3q724c_^)%@5irzmf=xz'"

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'seahub_db',
        'USER': 'seafile',
        'PASSWORD': 'Cj6yaRO#TWv6',
        'HOST': '127.0.0.1',
        'PORT': '3306'
    }
}


CACHES = {
    'default': {
        'BACKEND': 'django_pylibmc.memcached.PyLibMCCache',
        'LOCATION': '127.0.0.1:11211',
    },
    'locmem': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    },
}
COMPRESS_CACHE_BACKEND = 'locmem'

# EMAIL_USE_TLS                       = False
# EMAIL_HOST                          = 'localhost'
# EMAIL_HOST_USER                     = ''
# EMAIL_HOST_PASSWORD                 = ''
# EMAIL_PORT                          = '25'
# DEFAULT_FROM_EMAIL                  = EMAIL_HOST_USER
# SERVER_EMAIL                        = EMAIL_HOST_USER

SITE_ROOT                           = '/'
SITE_BASE                           = 'http://127.0.0.1'
SITE_NAME                           = 'Lnovus Dev Network'
SITE_TITLE                          = 'Private Cloud'

TIME_ZONE                           = 'Europe/Moscow'
LANGUAGE_CODE                       = 'ru'
LANGUAGES                           = (
    ('en', 'English'),
    ('ru', 'Russian'),
)


ENABLE_SIGNUP                       = False
ACTIVATE_AFTER_REGISTRATION         = False
SEND_EMAIL_ON_ADDING_SYSTEM_MEMBER  = False
SEND_EMAIL_ON_RESETTING_USER_PASSWD = False
CLOUD_MODE                          = False
ENABLE_WIKI                         = False

LOGIN_REMEMBER_DAYS                 = 1
LOGIN_ATTEMPT_LIMIT                 = 3

FILE_PREVIEW_MAX_SIZE               = 30 * 1024 * 1024
SESSION_COOKIE_AGE                  = 60 * 60 * 24 * 7 * 2
SESSION_SAVE_EVERY_REQUEST          = False
SESSION_EXPIRE_AT_BROWSER_CLOSE     = False

FILE_SERVER_ROOT                    = 'http://127.0.0.1/seafhttp'
```

# Start Seafile as service

Add configs for systemd:

For seafile.service:

`sudo nano /etc/systemd/system/seafile.service` 

```ini
[Unit]
Description=Seafile
# link to mysql.service or postgresql.service
After=network.target mysql.service

[Service]
# type can be one of: simple, exec, forking, oneshot, dbus, notify or idle
# more info https://www.freedesktop.org/software/systemd/man/systemd.service.html
Type=forking
ExecStart=/home/seafile/seafile-server-latest/seafile.sh start
ExecStop=/home/seafile/seafile-server-latest/seafile.sh stop
User=seafile
Group=seafile
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

For seahub.service:

`sudo nano /etc/systemd/system/seahub.service `

```
[Unit]
Description=Seafile hub
After=network.target seafile.service

[Service]
# type can be one of: simple, exec, forking, oneshot, dbus, notify or idle
# more info https://www.freedesktop.org/software/systemd/man/systemd.service.html
Type=forking
# change start to start-fastcgi if you want to run fastcgi
ExecStart=/home/seafile/seafile-server-latest/seahub.sh start 17101
ExecStop=/home/seafile/seafile-server-latest/seahub.sh stop
User=seafile
Group=seafile
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Stop all earlie running processes for seafile/seahub:
```bash
sudo -u seafile /home/seafile/seafile-pro-server-7.1.5/seafile.sh stop
sudo -u seafile /home/seafile/seafile-pro-server-7.1.5/seahub.sh stop

# kill all process
pkill -9 -u seafile
```

Start seafile as service:
```bash
# Перезапустим systemd
systemctl daemon-reload
# Запускаем сервис seafile
systemctl start seafile
# Enable the service on system boot
systemctl enable seafile
# Run seahub service
systemctl start seahub
# Enable the service on system boot
systemctl enable seahub

# check status
systemctl status seafile
systemctl status seahub
```

You can see the logs of the service using `journalctl -u wiki`

More info: https://seafile.gitbook.io/seafile-server-manual/deploying-seafile-under-linux/other-deployment-notes/start-seafile-at-system-bootup

Script for autoinstall: https://github.com/haiwen/seafile-server-installer

## Symlink: Для примонтированного диска

```
mv ../seafile-data/ /mnt/data/
ln -s /mnt/data/seafile-data/ /home/seafile/seafile-data
chown -R seafile:seafile /mnt/data/seafile-data/
```

## Add Fail2Ban

Info here: https://download.seafile.com/published/seafile-manual/security/fail2ban.md

## NGINX conf

`sudo nano /etc/nginx/nginx.conf`

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	# Максимальное количество соединений одного воркера.
	worker_connections 1024;
	# Метод выбора соединений.
	use epoll;
	# Принимать максимально возможное количество соединений.
	multi_accept on;
}

http {
	##
	# Basic Settings
	##

	# Метод отправки данных sendfile эффективнее чем read+write.
	sendfile on;

	# Ограничивает объём данных, который может передан за один вызов sendfile(). Нужно для исключения ситуации когда одно соединение может целиком захватить воркер.
	sendfile_max_chunk 128k;

	# Отправлять заголовки и начало файла в одном пакете.
	tcp_nopush on;
	tcp_nodelay on;

	# Сбрасывать соединение если клиент перестал читать ответ.
	reset_timedout_connection on;

	# Максимальный размер хэш-таблиц типов.
	types_hash_max_size 2048;

	# Отключить вывод версии nginx в ответе.
	server_tokens off;

	# Задание буфера для заголовка и тела запроса.
	client_header_buffer_size 2k;
	client_body_buffer_size 256K;

	# Ограничение на размер тела запроса.
	client_max_body_size 12m;

	# Количество и размер буферов для чтения большого заголовка запроса клиента.
	large_client_header_buffers 2 1k;

	# Таймаут при чтении тела запроса клиента.
	client_body_timeout 5;

	# Таймаут при чтении заголовка запроса клиента.
	client_header_timeout 3;

	# Таймаут, по истечению которого keep-alive соединение с клиентом не будет закрыто со стороны сервера.
	keepalive_timeout 60 60;

	# Таймаут при передаче ответа клиенту.
	send_timeout 3;

	# Ограничиваем число соединений с сервером с одного клиентского IP-адреса(limit_conn perip, находится в зоне server).
	# Позволяет ограничить число соединений по заданному ключу, в частности, число соединений с одного IP-адреса.
	#limit_conn_zone $binary_remote_addr zone=perip:5m;
	# Позволяет ограничить скорость обработки запросов по заданному ключу или, как частный случай, скорость обработки запросов, поступающих с одного IP-адреса.
	#limit_req_zone $binary_remote_addr zone=dynamic:5m rate=2r/s;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	# Поддерживаемые протоколы.
	ssl_protocols TLSv1.2; # Dropping SSLv3, ref: POODLE

	# Указывает, чтобы при использовании протоколов SSLv3 и TLS серверные шифры были более приоритетны, чем клиентские.
	ssl_prefer_server_ciphers on;

	# Наборы шифров.
	ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES256-SHA384;
	ssl_ecdh_curve secp384r1;

	# HSTS
	add_header Strict-Transport-Security "max-age=31536000";

	# Тип и объём кэша для хранения параметров сессий. Параметр shared задает общий для всех рабочих процессов nginx кэш.
	ssl_session_cache shared:SSL:10m;

	# Таймаут сессии в кэше.
	ssl_session_timeout 5m;

	#ssl_stapling on;
	#ssl_stapling_verify on;
	#resolver 83.217.24.42 8.8.8.8 valid=300s;
	#resolver_timeout 5s;

	##
	# Logging Settings
	##

	access_log off;
	#access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;
	gzip_comp_level 5;
	gzip_min_length 256;
	gzip_proxied any;
	gzip_vary on;
	gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
# 
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
```

## Скрипт для очистки сервера от мусора

Случается так, что по факту хранилище Seafile занимает больше места, чем указано в веб-интерфейсе администратора. Связано это с потерянными библиотеками и файлами и удалить их можно создав скрипт очистки и поместить его в cron.

В папке seafile создаем файл cleanup.sh `nano /home/seafile/seafile-server-latest/cleanup.sh` и добавляем следующее:

```bash
    #!/bin/bash
    # Остановка сервера Seafile
    echo Остановка сервера Seafile...
    systemctl stop seafile
    systemctl stop seahub
    #
    echo Ожидание остановки сервера
    sleep 60
    #
    # Запуск очистки от мусора
    echo Очистка Seafile от мусора...
    /home/seafile/seafile-server-latest/seaf-gc.sh
    #
    echo Очистка....
    sleep 60
    #
    # Запуск очистки от потерянных библиотек
    /home/seafile/seafile-server-latest/seaf-gc.sh -r
    echo Очистка....
    sleep 60
    #
    # Запуск сервера Seafile
    echo Запуск сервера Seafile...
    systemctl start seafile
    systemctl start seahub
    #
    echo Очистка завершена
```

Навастриваем задачу в cron:
```
crontab -e
```

Ставим выполнятся каждое воскресенье в 2 ночи (периодичность и время выполнения зависит от интенсивности использования сервера)

```
0 2 * * 0 /home/seafile/seafile-server-latest/cleanup.sh
```

## Кастомизация сервера

В настройках системы через веб-интерфейс присутствует возможность сменить логотип, а так же заставку экрана логина. Сменить же скин внешнего вида веб-интерфейса пользователей возможно только через CSS.

Для изменения необходимо создать файл custom.css и положить его и все сопутствующие файлы в папку /seahub-data/custom

Чтобы применить указанный CSS пропишите в seahub_settings.py следующее:

```
BRANDING_CSS = 'custom/custom.css'
```

## Administration
```
# stop services - look on command sequence
systemctl stop seahub
systemctl stop seafile

# start services
systemctl start seafile
systemctl start seahub
```

### Dark theme

```css
body {
    background-color: #202428;
}
.side-tabnav-tabs .tab a {
    display: block;
    font-size: 15px;
    padding: 4px 4px 4px 0;
    color: #D6D6D6;
    font-weight: normal;
}
.side-tabnav-tabs .tab-cur a, .side-tabnav-tabs .tab-cur a:hover {
    background-color: #4285D6;
}
.side-tabnav-tabs .tab a:hover {
    background-color: #6CAFEA;
    text-decoration: none;
}
.side-tabnav h3.hd, .side-tabnav .hd h3 {
    color: #4285D6;
    font: bold 16px Helvetica;
}
#header {
    background: #262B31;
    width: 100%;
    height: 53px;
    font-size: 14px;
    border-bottom: 3px solid #000;
    padding-bottom: 4px;
    z-index: 2;
}
th, td {
    padding: 5px 3px;
    border-bottom: 1px solid #000;
}
a {
    color: #4285D6;
    text-decoration: none;
    font-weight: bold;
}
#right-panel .hd, .tabnav, .repo-file-list-topbar, .commit-list-topbar, .file-audit-list-topbar, #dir-view .repo-op, .wiki-top {
    padding: 9px 10px;
    background: #121417;
    margin-bottom: .5em;
    border-radius: 2px;
}
h3 {
    font-size: 16px;
    color: #4285D6;
    font-weight: normal;
}
.side-nav-footer {
    padding: 12px 20px 16px;
    background: #121417;
    border-top: 0px solid #000;
    border-radius: 3px;
}
.side-nav {
    position: fixed;
    top: 52px;
    bottom: 0;
    background: #262B31;
    z-index: 1;
    padding: 20px;
    overflow: hidden;
    border-right: 3px solid #000;
}
.side-tabnav-tabs .tab [class^="sf2-icon-"] {
    display: inline-block;
    width: 42px;
    margin-right: 5px;
    text-align: center;
    vertical-align: middle;
    font-size: 24px;
    line-height: 1;
    color: #FEA717;
}
.op-icon.sf2-x, .op-icon.sf2-x:hover {
    color: #4285D6;
}
body, input, textarea, button, select {
    font: 13px/1.5 Arial,Helvetica,sans-serif;
    color: #FEA717;
    word-wrap: break-word;
}
.btn-white, .tabnav button, .repo-file-list-topbar .op-btn {
    height: 29px;
    background: #121417;
    line-height: 17px;
}
.hl {
    background-color: #202428;
}
.repo-op .op-link {
    display: inline-block;
    height: 30px;
    margin-right: 15px;
    color: #4285D6;
    font-size: 24px;
}
.grid-view-icon-btn.active, .list-view-icon-btn.active {
    color: #FEA717;
    background: #121417;
    cursor: default;
}
.grid-view-icon-btn, .list-view-icon-btn {
    display: inline-block;
    padding: 0 6px;
    height: 29px;
    background: #121417;
    border: 0px solid #ccc;
    font-size: 18px;
    color: #4285D6;
    cursor: pointer;
    border-radius: 0;
}
.grid-item .text-link {
    display: block;
    text-align: center;
    color: #737376;
    font-size: 14px;
    line-height: 17px;
}
.grid-item .text-link.hl {
    color: #4285D6;
    background: #202428;
}
.grid-item .img-link.hl {
    background: #4285D6;
}
.sf-popover {
    width: 240px;
    background: #202428;
    border: 0px solid #000;
    border-radius: 3px;
    box-shadow: 0 0 4px #000;
    position: absolute;
    z-index: 20;
}
.account-popup .item {
    display: block;
    padding: 8px 18px;
    border-bottom: 1px solid #000;
}
#traffic-stat {
    color: #4285D6;
    font-weight: normal;
}
.account-popup a.item {
    color: #737376;
    font-weight: normal;
}
.account-popup a.item:hover { background: #4285D6; text-decoration: none; color: #fff;
}
.search-form .search-submit {
    position: absolute;
    right: 1px;
    top: 1px;
    width: 30px;
    height: 24px;
    padding: 0;
    border: 0;
    margin: 0;
    background: #121417;
}
.search-input {
    margin: 0;
    background: #202428;
    border: 1px solid #000;
    border-radius: 3px;
}
body, input, textarea, button, select {
    font: 13px/1.5 Arial,Helvetica,sans-serif;
    color: #FEA717;
    word-wrap: break-word;
}
#search-form, #search-user-form, #search-repo-form {
    padding: 7px 5px;
    background: #262B31;
    border-radius: 2px;
    margin-bottom: 1.2rem;
}
#search-form .advanced-search {
    color: #FEA717;
    font-size: 16px;
    cursor: pointer;
    margin-left: 4px;
}
button, input[type=submit], input[type=button], input.submit, .sf-btn-link, .fileinput-button, select {
    padding: 5px 6px;
    background: -webkit-linear-gradient(top,#121417,#121417);
    background: -moz-linear-gradient(top,#464647,#000);
    background: linear-gradient(top,#fafafb,#eee);
    border: 0px solid #c5c5c5;
    border-radius: 2px;
}
#notifications .sf2-icon-bell {
    font-size: 24px;
    line-height: 1;
    color: #FEA717;
}
#notice-popover li {
    padding: 9px 0 3px;
    border-bottom: 1px solid #121417;
}
.sf-popover-hd {
    padding: 5px 0 3px;
    border-bottom: 1px solid #121417;
    margin: 0 10px;
}
.up-outer-caret .inner-caret {
    border-bottom-color: #4285D6;
    top: 0px;
    left: -10px;
}
#my-info {
    cursor: pointer;
    color: #FEA717;
}
#simplemodal-container {
    padding: 20px;
    background-color: #262B31;
    -moz-border-radius: 4px;
    border-radius: 4px;
    -webkit-box-sizing: content-box;
    box-sizing: content-box;
    box-shadow: 0 0 4px #000;
}
.input, .textarea {
    width: 260px;
    padding: 2px 3px;
    border-radius: 2px;
    border-color: #000;
    margin-bottom: 5px;
    background: #202428;
}
#notice-popover .avatar {
    border-radius: 3px;
    float: left;
}
#account .avatar {
    vertical-align: middle;
    border-radius: 3px;
}
.side-textnav-tabs .tab a {
    display: block;
    padding: 10px 0;
    font-weight: normal;
    color: #999;
    border-bottom: 1px solid #000;
    margin-bottom: 3px;
}
.setting-item h3 {
    color: #000;
    padding-bottom: .2em;
    border-bottom: 1px solid #000;
    margin-bottom: .7em;
}
.setting-item h3 {
    color: #4285D6;
    padding-bottom: .2em;
    border-bottom: 1px solid #000;
    margin-bottom: .7em;
}
#lang-context-selector {
    position: absolute;
    top: 60px;
    border: 1px solid #000;
    background: #202428;
    padding: 5px 0;
    box-shadow: 0 2px 4px #000;
    white-space: nowrap;
    z-index: 10;
}
#lang-context-selector a {
    color: #999999;
    display: block;
    padding: 1px 5px;
    font-weight: normal;
}
.file-tree-cont, .dir-tree-cont {
    padding: 5px;
    height: 280px;
    overflow: auto;
    border: 1px solid #000;
    margin: 5px 0 10px;
}
td {
    color: #9C9C9C;
    font-size: 14px;
    word-break: break-all;
}
th {
    text-align: left;
    font-weight: normal;
    color: #D6D6D6;
}
.side-search-form .input {
    width: 188px;
    padding: 2px 5px;
    background: #202428;
    box-shadow: inset 0 1px 1px #000;
}
.new-narrow-panel .hd {
    color: #ccc;
    font-size: 16px;
    padding: 5px 20px;
    background: #121417;
    border-bottom: 1px solid #000;
}
.new-narrow-panel {
    width: 388px;
    border: 1px solid #000;
    border-radius: 4px;
    box-shadow: 0 3px 2px #000;
    margin: 5em auto;
}
h2 {
    font-size: 1.5em;
    color: #4285D6;
    font-weight: bold;
}
.empty-tips {
    padding: 30px 40px;
    background-color: #262B31;
    border: solid 1px #000;
    border-radius: 3px;
    box-shadow: inset 0 0 0px #000;
    margin-top: 5.5em;
}
.left-right-tabs-nav .ui-state-active a, .left-right-tabs-nav .ui-state-active a:hover {
    background: #4285D6;
    color: #fff;
}
.select2-results .select2-no-results, .select2-results .select2-searching, .select2-results .select2-ajax-error, .select2-results .select2-selection-limit {
    background: #202428;
    display: list-item;
    padding-left: 5px;
    
}
.select2-container-multi .select2-choices .select2-search-field {
    margin: 0;
    padding: 0;
    white-space: nowrap;
    background: #202428;
}
.select2-container-multi .select2-choices {
    border-color: #000;
    border-radius: 2px;
    background-image: none;
    background: #202428;
}
.select-white, .perm-add-perm, .user-perm-add-perm, .group-perm-add-perm, .perm-toggle-select, .folder-perm-select, .share-permission-select, .user-role-select, .user-status-select {
    position: relative;
    padding: 3px 2px;
    background: #202428;
    border: 1px solid #000;
    border-radius: 2px;
}
.select2-drop {
    margin-top: -1px;
    background: #202428;
    color: #999;
    border: 1px solid #000;
    border-top: 0;
    border-radius: 0 0 4px 4px;
    -webkit-box-shadow: 0 4px 5px rgba(0,0,0,.15);
    box-shadow: 0 4px 5px rgba(0,0,0,.15);
}
.select2-container-multi .select2-choices .select2-search-choice {
    background: #121417;
    padding: 3px 5px 3px 18px;
    margin: 3px 0 3px 5px;
    position: relative;
    line-height: 13px;
    color: #FEA717;
    cursor: default;
    border: 1px solid #000;
    border-radius: 3px;
    -webkit-box-shadow: 0 0 2px #fff inset,0 1px 0 rgba(0,0,0,0.05);
    box-shadow: 0 0 0px #000;
    background-clip: padding-box;
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
h4 {
    font-size: 1.1em;
    color: #4285D6;
    font-weight: normal;
}
.seahub-web-settings h4 {
    padding: 3px 10px;
    background: #262B31;
    border-radius: 2px;
    margin: 15px 0;
    font-color;
}
.event-item {
    padding: 7px 8px;
    border-bottom: 1px solid #000;
}
.activity-group-hd {
    padding: 8px;
    color: #CCCCCC;
    font-size: 16px;
    border-bottom: 3px solid #000;
    margin: 0;
}
#ls-ch h2, #ls-ch h3 {
    margin: 0 0 .2em;
    color: #CCC;
    font-weight: bold;
}
#ls-ch .commit-time, .commit-time {
    color: #CCC;
    font-size: 13px;
    font-weight: normal;
    margin-top: 0;
}
.article {
    padding: 40px 200px 40px 60px;
    font-size: 14px;
    line-height: 1.6;
    color: #ccc;
}
.wiki-nav .link {
    color: #A3A3A3;
    font-weight: normal;
}
.sf-btn-link {
    display: inline-block;
    color: #a3a3a3;
    line-height: 19px;
    text-decoration: none;
    font-weight: normal;
}
.messages .success {
    padding: 5px;
    background: #121417;
    margin: 0;
}
.fixed-upload-file-dialog .hd {
    padding: 6px 10px;
    background: #121417;
    margin: 0;
}
.fixed-upload-file-dialog .con {
    height: 164px;
    overflow-y: auto;
    background: #262B31;
}
.fixed-upload-file-dialog {
    width: 540px;
    position: fixed;
    bottom: 0;
    right: 10px;
    background: #262B31;
    border: 1px solid #000;
    border-radius: 3px;
    box-shadow: 0 0 6px #000;
}
.ui-progressbar, .progress {
    background-color: #202428;
    background-image: linear-gradient(to bottom,#202428,#202428);
    background-repeat: repeat-x;
    border-radius: 4px 4px 4px 4px;
    box-shadow: 0 1px 2px rgba(0,0,0,0.1) inset;
    height: 1em;
    overflow: hidden;
}
.ui-progressbar-value, .progress .bar {
    background: #FEA717;
    height: 100%;
}
.sf-dropdown-menu {
    position: absolute;
    background: #202428;
    padding: 6px 1px;
    margin: 2px 0 0;
    border: 1px solid #000;
    border-radius: 3px;
    box-shadow: 0 2px 3px 0 #000;
    z-index: 10;
}
.sf-dropdown-menu li a, .sf-dropdown-menu a {
    display: block;
    padding: 4px 12px;
    min-width: 110px;
    white-space: nowrap;
    color: #ccc;
    font-weight: normal;
}
.sf-dropdown-menu a:hover { background: #4285D6; text-decoration: none; color: #fff;
}
input[type=text], input[type=password] {
    box-sizing: content-box;
    height: 22px;
    background: #202428;
    
}
.repo-folder-perm-folder-path {
    width: 150px;
    padding: 3px 33px 3px 5px;
    border: 1px solid #000;
    border-radius: 2px;
    margin: 0;
}
.repo-folder-perm-add-folder {
    position: absolute;
    right: 0;
    top: 1px;
    width: 29px;
    height: 28px;
    line-height: 28px;
    text-align: center;
    font-size: 14px;
    color: #ccc;
    cursor: pointer;
    border-left: 1px solid #000;
    margin: 0;
}
dt {
    color: #ccc;
    margin: 24px 0 2px;
    font-weight: normal;
}
dd {
    margin-bottom: .8em;
    color: #FEA717;
}
#share-popup .show-or-hide-password, #add-user-form .show-or-hide-password {
    right: 36px;
    background: #121417;
}
#share-popup .generate-random-password, #add-user-form .generate-random-password {
    background: #121417;
}


.login-panel {
    background: #333;
    border-radius: 3px;
    padding: 20px 60px;
    width: 390px;
    margin: 0 auto;
    box-shadow: 0 0 8px #a7a6a9;
}

.header-bar {
    padding: 9px 10px;
        padding-bottom: 9px;
    background: #202428;
    margin-bottom: .5em;
    border-radius: 2px;
    padding-bottom: 0;
    height: 48px;
    overflow: hidden;
}
```


Дополнительные настройки представлены здесь: https://open-networks.ru/d/34-seafile-ce-nastroyka-oblachnogo-khranilishcha
Integrate Office: https://seafile.gitbook.io/seafile-server-manual/deploy-seafile-pro-edition/online-file-preview-and-edit/office-online-server-integration
