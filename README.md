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

## Установка seafile pro 

```
# установка СУБД MySQL
# устанавливаем и настраиваем базу данных MariaDB Server
sudo apt-get -y install mariadb-server
# перезапускаем сервер баз данных
sudo systemctl restart mysql

# установим пароль суперпользователя MySQL
sudo mysql -u root
use mysql;
UPDATE user set password=PASSWORD("bbByz56h@l$b") where User='root';
FLUSH PRIVILEGES;
update user set plugin='' where User='root';
exit;
# перезапускаем сервер баз данных
systemctl restart mariadb
# теперь настроим безопасное подключение к БД
sudo mysql_secure_installation 	# только на смену пароля отвечаем No
# перезапускаем сервер баз данных
systemctl restart mariadb

# теперь получить доступ к серверу БД можно только по паролю
sudo mysql -uroot -p

# seafile во время установки запросит пароль от root и установит требуемые БД

# проверяем установку базы данных
sudo service mysql status	# sudo service mysql start|restart|stop

# подключение можно выполнять так
mysql -u <имя_пользователя> -p	

# установка дополнительных пакетов
sudo apt-get update
sudo apt-get install python3 \
	python3-setuptools \
	python3-pip \
	python3-pil \
	python3-mysqldb -y 
	
sudo apt-get install -y libmemcached-dev zlib1g-dev libssl-dev python-dev build-essential

pip install --timeout=3600 Pillow pylibmc captcha jinja2 sqlalchemy django-pylibmc django-simple-captcha python3-ldap

pip install --timeout=3600 Pillow 
pip install --timeout=3600 pylibmc 
pip install --timeout=3600 captcha 
pip install --timeout=3600 jinja2 
pip install --timeout=3600 sqlalchemy 
pip install --timeout=3600 django-pylibmc 
pip install --timeout=3600 django-simple-captcha 
pip install --timeout=3600 python-ldap

pip3 install --timeout=3600 Pillow 
pip3 install --timeout=3600 pylibmc 
pip3 install --timeout=3600 captcha 
pip3 install --timeout=3600 jinja2 
pip3 install --timeout=3600 sqlalchemy 
pip3 install --timeout=3600 django-pylibmc 
pip3 install --timeout=3600 django-simple-captcha 
pip3 install --timeout=3600 python3-ldap

# установка openjdk
sudo su	# [root]
sudo nano /etc/apt/sources.list
# add this: deb http://ftp.us.debian.org/debian sid main
sudo apt-get update
sudo apt-get install openjdk-8-jre
# delete string in /etc/apt/sources.list
sudo update-alternatives --config java	# дополнительно
# show version
java -version

# Install poppler-utils
sudo apt-get install poppler-utils
# Install Python libraries
sudo pip install boto
sudo pip install setuptools --upgrade

# скачать и распаковать пакет
cd /opt
sudo wget https://download.seafile.com/d/6e5297246c/files/?p=%2Fpro%2Fseafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz
tar xf seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz
mkdir /home/seafile
mv /opt/seafile/seafile-pro-server-7.1.5 /home/seafile/seafile-pro-server-7.1.5
cd /home/seafile/seafile-pro-server-7.1.5
# директории перед установкой будут выглядеть так
<main_dir>
├── seafile-license.txt
└── seafile-pro-server-7.1.5/
# откроем порт
sudo ufw allow 11317/tcp
# запускаем установку
sudo su
./setup-seafile-mysql.sh 

# запускаем как сервис через системного демона
# Create a new file named seafile.service inside directory /etc/systemd/system
nano /etc/systemd/system/seafile.service

# Paste the following contents
[Unit]
Description=Seafile hub
After=network.target seafile.service

[Service]
# change start to start-fastcgi if you want to run fastcgi
Environment="LC_ALL=C"
ExecStart=/home/seafile/seafile-server-latest/seahub.sh start-fastcgi
ExecStop=/home/seafile/seafile-server-latest/seahub.sh stop
User=seafile
Group=seafile
Type=oneshot
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

# Создадим пользователя
sudo adduser seafile
sudo usermod -aG sudo seafile
# Дадим ему права
chown -R seafile:seafile /home/seafile
# Перезапустим systemd
systemctl daemon-reload
# Запускаем сервис
systemctl start seafile
# Enable the service on system boot
systemctl enable seafile
```

You can see the logs of the service using `journalctl -u wiki`

При необходимости, также выставить права 666 на package.json


./seafile.sh start 
ufw allow 11318/tcp
./seahub.sh start 11318 

https://github.com/haiwen/seafile-server-installer


# Install seafile like script

```bash


# -------------------------------------------
# Additional requirements
# -------------------------------------------
apt-get update

apt-get install -y python3 python3-setuptools python3-pip python3-ldap memcached openjdk-8-jre \
    libmemcached-dev libreoffice-script-provider-python libreoffice pwgen curl nginx

pip3 install --timeout=3600 Pillow pylibmc captcha jinja2 sqlalchemy psd-tools \
    django-pylibmc django-simple-captcha


service memcached start

# -------------------------------------------
# Setup Nginx
# -------------------------------------------

rm /etc/nginx/sites-enabled/*
nano /etc/nginx/sites-available/seafile.conf
ln -sf /etc/nginx/sites-available/seafile.conf /etc/nginx/sites-enabled/seafile.conf
service nginx restart

# -------------------------------------------
# MariaDB
# -------------------------------------------

apt-get install -y mariadb-server
service mysql start

mysqladmin -u root password <root-password>

# -------------------------------------------
# Seafile
# -------------------------------------------

mkdir -p /opt/seafile/installed
cd /opt/seafile
mv /opt/seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz /opt/seafile/seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz
tar xzf seafile-pro-server_7.1.5_x86-64_Ubuntu.tar.gz

useradd --system --comment "seafile" seafile --home-dir  /opt/seafile

cd /opt/seafile/seafile-pro-server-7.1.5

mkdir -p /opt/seafile/seafile-pro-server-7.1.5/conf

# -------------------------------------------
# Create ccnet, seafile, seahub conf using setup script
# -------------------------------------------

./setup-seafile-mysql.sh auto -u seafile -w <mysql-seafile-password> -r <mysql-root-password>

# -------------------------------------------
# Configure Seafile WebDAV Server(SeafDAV)
# -------------------------------------------

# -------------------------------------------
# Configuring seahub_settings.py
# -------------------------------------------

chown seafile:seafile -R /opt/seafile

PRO_PY=${INSTALLPATH}/pro/pro.py
python /opt/seafile/seafile-pro-server-7.1.5/pro/pro.py setup --mysql --mysql_host=127.0.0.1 --mysql_port=3306 --mysql_user=seafile --mysql_password=<mysql-seafile-password> --mysql_db=seahub_db

# kill all process
pkill -9 -u seafile

# -------------------------------------------
# Fix permissions
# -------------------------------------------
chown seafile:seafile -R /opt/seafile
chown seafile:seafile -R /tmp/seafile-office-output/

# -------------------------------------------
# Start seafile server
# -------------------------------------------
service seafile-server start	# using initial script


sudo -u seafile /opt/seafile/seafile-pro-server-7/seafile.sh start
sudo -u seafile /opt/seafile/seafile-pro-server-7/seahub.sh start

```

# Seafile NGINX config

Add to `/etc/nginx/sites-available/seafile.conf`

```NGINX
log_format seafileformat '$http_x_forwarded_for $remote_addr [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $upstream_response_time';

server {
    listen 80;
    server_name seafile.example.com;

    proxy_set_header X-Forwarded-For $remote_addr;

    location / {
         proxy_pass         http://127.0.0.1:8000;
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
         proxy_pass http://127.0.0.1:8082;
         client_max_body_size 0;
         proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_connect_timeout  36000s;
         proxy_read_timeout  36000s;

        access_log      /var/log/nginx/seafhttp.access.log seafileformat;
        error_log       /var/log/nginx/seafhttp.error.log;
    }
    location /media {
        root /opt/seafile/seafile-server-latest/seahub;
    }
    location /seafdav {
        proxy_pass         http://127.0.0.1:8080/seafdav;
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

Затем:
```
ln -sf /etc/nginx/sites-available/seafile.conf /etc/nginx/sites-enabled/seafile.conf
service nginx restart
```

# Seafile configs [/opt/seafile/conf]

## ccnet.conf

```conf
[General]
SERVICE_URL = http://127.0.0.1:8000

[Database]
ENGINE = mysql
HOST = 127.0.0.1
PORT = 3306
USER = seafile
PASSWD = Fee3eena
DB = ccnet_db
CONNECTION_CHARSET = utf8
```

## gunicorn.conf.py

```python
import os

daemon = True
workers = 5

# default localhost:8000
bind = "127.0.0.1:8000"

# Pid
pids_dir = '/opt/seafile/pids'
pidfile = os.path.join(pids_dir, 'seahub.pid')

# for file upload, we need a longer timeout value (default is only 30s, too short)
timeout = 1200

limit_request_line = 8190
```

## seafdav.conf

```
[WEBDAV]
enabled = true
port = 8080
fastcgi = true
share_name = /seafdav
```

## seafevents.conf

```
[DATABASE]
type = mysql
host = 127.0.0.1
port = 3306
username = seafile
password = Fee3eena
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

[SEAHUB EMAIL]
enabled = true

## interval of sending Seahub email. Can be s(seconds), m(minutes), h(hours), d(days)
interval = 30m

# Enable statistics
[STATISTICS]
enabled=true
```

