# MiniDevLab on Linux (local network dev host)

update ssh on windows for connection with this manual
```
https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH
```

## Первоначальная настройка
### Настройка репозиториев на Debian 10 buster
```bash
# редактируем файл sources.list (добавляем contrib non-free для каждого репозитория)
sudo nano /etc/apt/sources.list

# Следует привести файл /etc/apt/sources.list к следующему виду
deb http://deb.debian.org/debian/ buster main contrib non-free
deb-src http://deb.debian.org/debian/ buster main contrib non-free

deb http://security.debian.org/debian-security buster/updates main contrib
deb-src http://security.debian.org/debian-security buster/updates main contrib

deb http://deb.debian.org/debian/ buster-updates main contrib non-free
deb-src http://deb.debian.org/debian/ buster-updates main contrib non-free

deb http://deb.debian.org/debian/ buster-backports main contrib non-free
deb-src http://deb.debian.org/debian/ buster-backports main contrib non-free

# сохраняем изменения: Ctrl+X затем Y и Enter
# обновим зависимости
sudo apt update
sudo apt -y dist-upgrade
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
sudo apt-get install gcc g++ make
sudo apt-get update && sudo apt-get install yarn
sudo apt-get install -y nodejs
# обновляем зависимости и пакеты
apt update
# проверяем версии nodejs и npm
nodejs -v && node -v && npm -v

# включаем FireWall
ufw enable
# просмотрим установленные правила firewall 
ufw status

# переходим в директорию по умолчанию
cd ~
# Установим дополнительные пакеты certbot для SSL/TLS
sudo apt install -y certbot python3-certbot-nginx


# устанавливаем дополнительные пакеты pip
sudo apt-get install -y python3-pip
sudo apt-get install -y python-pip
# upgrade pip version
python -m pip install --user art610 --upgrade pip
python3 -m pip install --user art610 --upgrade pip
python3 -m pip install virtualenv
python3 -m pip install virtualenvwrapper
sudo apt-get -y install python3-dev


# очистим старые записи после обновления
hash -d pip
# проверим версии python и pip
python -V && pip -V
python3 -V && pip3 -V

# if we have problem with pip when we should use cmd python -m pip
# add alias for this 
# open ~/.bash_aliases
nano ~/.bash_aliases
# add this
alias pip3='python3 -m pip'
alias pip='python -m pip'
# open ~/.bashrc
nano ~/.bashrc
# add this
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# обновляем зависимости
sudo apt update

# выводим основную информацию
cat /etc/os-release    # информация о релизе ОС
arch    # архитектура ОС
nodejs -v && node -v && npm -v
git --version && python3 -V && pip3 -V
```

### Доступ из локальной сети

```
# смотрим сетевые интерфейсы
ip a 
# редактируем настройки сети
sudo nano /etc/network/interfaces
# дописываем нужные параметры сети
auto enp3s2
        iface enp3s2 inet static
        address 17.10.1.1
        netmask 255.255.255.0
	# gateway 17.10.1.2  
	# network 192.168.1.0
	# broadcast 192.168.1.1
	# dns-nameservers 8.8.8.8 8.8.4.4

# перезапускаем сеть
systemctl restart networking
# проверяем настройки сети
ip a
# если интерфейс не поднялся, то выполняем
sudo ifup enp3s2
```

### Install DHCP server

```
# Для начала установите DHCP-сервер на Debian
sudo apt-get install -y isc-dhcp-server
# выбрать сетевую карту, на которую привяжем DHCP
ip a
# edit config for dhcp-server
nano /etc/default/isc-dhcp-server
# add this
INTERFACESv4="enp3s2"
INTERFACESv6="

# Настройка сервера DHCP 
# Скопируем конфиг настроек (backup)
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak
# создадим конфигурацию с нашими всеми настройками
nano /etc/dhcp/dhcpd.conf
# add this
ddns-update-style none;
# не забудьте изменить данное значение:
option domain-name "dyn.lnovus.local";       
# вы можеет использовать dns серверы для dhcp клиентов:
option domain-name-servers 17.10.1.1, 8.8.8.8, 8.8.4.4;
# основной шлюз (сервер, через который они смогут попасть в инет или другую сеть) для dhcp $
option routers 17.10.1.1;
# broadcast адрес - не меняйте, если не знаете что это такое.
option broadcast-address 17.10.1.255;
# ntp серверы для dhcp клиентов.
option ntp-servers 17.10.1.1;
default-lease-time 86400;
max-lease-time 86400;
authoritative;
# log-facility local7;

subnet 17.10.1.0 netmask 255.255.255.0 {
        range 17.10.1.1 17.10.1.100;
}


# find and kill processes and pids for dhcp if exists
ps ax | grep dhcp
kill <process_id>
rm /var/run/dhcpd.pid

# restart dhcp
service isc-dhcp-server restart
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
apt update
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

# [if error] на Windows переходим в папку C:/Windows/System32/OpenSSH, где расположен ssh-client
# при необходимости в Параметр -> Приложения -> Доп. компоненты установим ssh-клиента
# если при вызове ssh-agent происходит ошибка, то выполняем в PowerShell
Set-Service ssh-agent -StartupType Manual

# теперь созданный ключ надо установить на сервер
# можно использовать такую команду
cat ~/.ssh/id_rsa.pub | ssh <username>@host_id 'cat >> ~/.ssh/authorized_keys'
# либо при наличии утилиту
ssh-copy-id -i $HOME/.ssh/id_rsa.pub <username>@host_id
# либо самостоятельно зайти на сервер и создать файл authorized_keys
exit	# work like user
mkdir ~/.ssh
sudo nano ~/.ssh/authorized_keys	# like user
# затем в файл нужно скопировать данные открытого ключа id_rsa.pub 
# далее установим права
chmod 700 ~/.ssh/
sudo chmod 644 ~/.ssh/authorized_keys
# if needed: chown -R <username>:<username> /home/<username>/.ssh/
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

## Inner DNS Server - Local Domains Resolver





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
apt-get install -y python3 python3-setuptools python3-pip python3-ldap memcached libmemcached-dev libreoffice-script-provider-python libreoffice pwgen
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
    server_name 17.10.1.1;	# add server ip or domain here

    proxy_set_header X-Forwarded-For $remote_addr;

    location / {
         proxy_pass         http://127.0.0.1:17101;
         proxy_set_header   Host $host:$proxy_port;
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
        proxy_set_header   Host $host:$proxy_port;
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

Дополнительные настройки представлены здесь: https://open-networks.ru/d/34-seafile-ce-nastroyka-oblachnogo-khranilishcha
Integrate Office: https://seafile.gitbook.io/seafile-server-manual/deploy-seafile-pro-edition/online-file-preview-and-edit/office-online-server-integration

# Install JupyterLab

Using pip:
```
# install and upgrade jupyterlab
pip3 install jupyterlab
pip3 install jupyterlab --upgrade
# install additional packages
pip3 install --trusted-host pypi.org --trusted-host files.pythonhosted.org pandas
# run jupyter lab
jupyter lab
# add systemd service for autorun
sudo nano /etc/systemd/system/jupyter.service

# =============================================================
# add this to config
# =============================================================
[Unit]
Description=Jupyter Lab     
After=syslog.target network.target

[Service]
Type=oneshot    # or forking / simple
ExecStart=/usr/local/bin/jupyter lab --ip 127.0.0.1 --port 9090
ExecStart=/usr/local/bin/jupyter lab stop
WorkingDirectory=/home/jupyter/
User=art610
RemainAfterExit=yes
Restart=on-failure
RestartSec=10

[Install]
WantedBy=default.target         # or multi-user
# =============================================================

# activate config and run
systemctl daemon-reload
systemctl start jupyter
systemctl enable jupyter

# make home dir for workspace
mkdir /home/jupyter
chown -R art610:art610 /home/jupyter

# restart 
systemctl restart jupyter

# generate config and add password
jupyter lab --generate-config -u art610
```

