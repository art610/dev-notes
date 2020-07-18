# MiniDevLab on Linux (local network dev host)

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

## Установка DHCP и DNS сервера DNSMasq 

Разрешение трафика на локальном узле
```
sudo iptables -A INPUT -i <eth0> -j ACCEPT	# указываем нужное имя интерфейса
sudo /sbin/iptables-save	# сохраняем правила
apt-get install iptables-persistent	# установим для автоподгрузки файлов после преезапуска системы
sudo ufw allow bootps		
```

More info: https://itproffi.ru/nastrojka-pravil-iptables-v-linux/

Если получили ошибку "unable to resolve host", то меняем localhost в файле /etc/hosts:
```
# получаем текущий hostname
hostname
# редактируем /etc/hosts
nano /etc/hosts
# добавляем hostname примерно так
127.0.0.1       debian-local-dev
```

More info: https://losst.ru/oshibka-sudo-unable-to-resolve-host

Настраиваем DNSMasq:

```
# install
apt install dnsmasq
# edit hosts-file
nano /etc/hosts
# add this
# /etc/hosts на компьютере-шлюзе 
127.0.0.1 localhost 
192.168.0.1 mars mars.sol
# Если возникнут проблемы, проверьте конфигурацию брандмауэра! 

# Конфигурация dnsmasq выполняется в файле /etc/dnsmasq.conf
nano /etc/dnsmasq.conf
# add this configs
domain-needed
bogus-priv
interface=enp3s2
strict-order

# DHCP config
dhcp-range=17.10.1.1,17.10.1.100,7D
dhcp-option=3,17.10.1.1
dhcp-option=1,255.255.255.0
dhcp-authoritative
# dhcp-boot=pxelinux.0,17.10.1.1

# DNS config
local=/sol/
domain=sol
expand-hosts

# edit /etc/dhcp/dhclient.conf
nano /etc/dhcp/dhclient.conf
# uncomment this string
prepend domain-name-servers 127.0.0.1;

# edit /etc/resolv.conf
nano /etc/resolv.conf
# add nameservers in strict order
nameserver 127.0.0.1
nameserver 192.168.8.1
# nameserver 8.8.8.8
# nameserver 8.8.4.4

# restart dnsmasq and networks
sudo systemctl restart networking
/etc/init.d/dnsmasq restart
```

More info: https://modx.cc/linux/programma-dnsmasq-(dhcp-i-server-imen)/

Now, if we on client open terminal/cmd and write `ping mars.sol`, we get request from 17.10.1.1. If we use some web-servers like nginx, we can open in browser url: http://mars.sol [17.10.1.1].

```
# check in cmd
ping mars.sol
nslookup mars.sol
```

We can edit our /etc/hosts on server and add domains for our ip's.


### Доступ по SSH

Обнович ssh на windows для подключения с последней версией протокола:

Manual here: https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH

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

### Локальный Certificate Authority и сертификаты SSL с помощью OpenSSL

```bash
# создаем директорию под сертификаты и переходим в нее
mkdir /home/certs
cd /home/certs
# выпускаем корневой сертификат
openssl genrsa -out rootCA.key 4096
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem
# в итоге мы получим корневой сертификат rootCA.pem и ключ root.CA.key

# создаем скрипт для выпуска самоподписанных сертификатов
nano create_certificate_for_domain.sh
```

**В скрипт добавляем следующее:**

```bash
#!/bin/bash
# скрипт для генерации сертификатов
if [ -z "$1" ]
then
  echo "Please supply a subdomain to create a certificate for";
  echo "e.g. mysite.localhost"
  exit;
fi

if [ -f device.key ]; then
  KEY_OPT="-key"
else
  KEY_OPT="-keyout"
fi

DOMAIN=$1
COMMON_NAME=${2:-$1}

SUBJECT="/C=CA/ST=None/L=NB/O=None/CN=$COMMON_NAME"
NUM_OF_DAYS=999
openssl req -new -newkey rsa:4096 -sha256 -nodes $KEY_OPT device.key -subj "$SUBJECT" -out device.csr

cat v3.ext | sed s/%%DOMAIN%%/$COMMON_NAME/g > /tmp/__v3.ext

openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days $NUM_OF_DAYS -sha256 -extfile /tmp/__v3.ext

mkdir ./$DOMAIN
mv device.key ./$DOMAIN/device.key
mv device.csr ./$DOMAIN/$DOMAIN.csr
cp device.crt ./$DOMAIN/$DOMAIN.crt

# remove temp file
rm -f device.crt;
```

Далее создаем файл с настройками v3.ext 
`nano /home/certs/v3.ext`

Добавляем в него домены, для которых валиден сертификат и др.
```bash
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = %%DOMAIN%%
DNS.2 = *.%%DOMAIN%%
```


Делаем скрипт исполняемым и запускаем:
```
chmod a+x create_certificate_for_domain.sh
./create_certificate_for_domain.sh mars.sol
```

Получаем два файла: mars.sol.crt и device.key

Ковертируем rootCA.pem в rootCA.crt
```
openssl x509 -outform der -in rootCA.pem -out rootCA.crt
```

Далее редактируем конфиг NGINX для SSL, добавляя в файл /etc/nginx/sites-available/ следующие строки и перенаправление с 443-го порта, незабывая открыть порт в настройках сетевого экрана:
```nginx
 listen 443;
        ssl on;
        server_name mars.sol;
        ssl_certificate /home/certs/mars.sol.crt;
        ssl_certificate_key /home/certs/device.key;
```

Теперь можно добавить корневой сертификат в доверенные для браузеров, приложений и систем, чтобы работать по https с валидным сертификатом.
Это можно сделать через "Менеджер сертификатов" в браузере, свойствах обозревателя и т.п. Для Ubuntu и Debian, их командной строки, добавить сертификат можно так:
```
apt-get install ca-certificates
sudo mkdir /usr/share/ca-certificates
sudo mkdir /usr/share/ca-certificates/extra
sudo cp /home/certs/rootCA.crt /usr/share/ca-certificates/extra/rootCA.crt
sudo dpkg-reconfigure ca-certificates
sudo update-ca-certificates

# если браузер использует свою базу данных сертификатов, делаем так
sudo apt-get install certutil
certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n "My Homemade CA" -i /path/to/CA/rootCA.pem
```

В MacOSX можно использовать приложение Keychain Access. 

Более подробно здесь: https://habr.com/ru/post/352722/

Добавление сертификата в доверенные в Linux: https://unix.stackexchange.com/questions/90450/adding-a-self-signed-certificate-to-the-trusted-list


### Wake-on-LAN

Manuals:
https://www.opennet.ru/tips/2503_lan_linux_ethernet_boot.shtml
http://pyatilistnik.org/kak-nastroit-wake-on-lan-v-linux/
https://habr.com/ru/post/77191/
https://mywebpc.ru/windows/wake-on-lan-v-windows/
https://userello.ru/internet/chto-takoe-wake-lan-i-kak-ego-vklyuchit

### Отключение графического интерфейса сервера

```
# переходим в консоль по Ctrl+Alt+F1
# установим aptitude если не установлено
sudo apt-get install aptitude
# определим имя графического менеджера
aptitude search '~i~Px-display-manager'
# перейдем в многопользовательский режим 
systemctl set-default multi-user.target
# остановим менеджер графики
sudo service lightdm stop 	# управление: <stop | start | restart> 
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

Добавим отдельного пользователя:
```
# создадим нового пользователя
useradd --system --comment "WikiJS User" wikiuser --home-dir  /home/wiki -m -U -s /bin/false
chown wikiuser:wikiuser -R /home/wiki
# зададим пароль пользователя
passwd wikiuser	# 1@7OrIX7Fp6v
# добавим пользователя в группу nginx
usermod -a -G wikiuser www-data
```

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

# Restart=always
# Consider creating a dedicated user for Wiki.js here:
User=wikiuser
Environment=NODE_ENV=production
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

### Run WikiJS using NGINX/HTTPS

Для запуска через NGINX мы должны проксировать запросы с домена, привязанного к ip сервера на 17.10.1.1:3000
Для этого создадим конфиг в NGINX:
`nano /etc/nginx/sites-available/wikijs.conf`

```nginx
server {
        listen 80;
        server_name wiki.ln;
        return 302 https://$host$request_uri;
}

server {
server {
        listen [::]:443 ssl http2;
        listen 443 ssl http2;
        ssl on;
        server_name wiki.ln;

        charset utf-8;
        client_max_body_size 250M;
        error_log /var/log/nginx/error.log;

        ssl_certificate_key /home/certs/wiki.ln/device.key;
        ssl_certificate /home/certs/wiki.ln/wiki.ln.crt;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/nginx/dhparam.pem;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
        add_header Strict-Transport-Security max-age=15768000;
        server_tokens off;

        location / {
                proxy_pass http://17.10.1.1:3000;
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Host $server_name;
                proxy_set_header   X-Forwarded-Proto https;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
                proxy_next_upstream error timeout http_502 http_503 http_504;
    }
}
```

Активируем конфиг и перезапустим NGINX:
```
ln -s /etc/nginx/sites-available/wikijs.conf /etc/nginx/sites-enabled/wikijs.conf
service nginx restart
```



## Установка Jira & Confluence

Создание базы данных в postgres:
```
su - postgres
psql
# создание базы данных под confluence и пользователя для atlassian продуктов
CREATE DATABASE confluencedb with encoding='UNICODE';
CREATE USER atlassianusr with password 'add-user-password';
GRANT ALL PRIVILEGES ON DATABASE confluencedb TO atlasianusr;
# создание базы данных под jira
CREATE DATABASE jiradb with encoding='UNICODE';
GRANT ALL PRIVILEGES ON DATABASE jiradb TO atlasianusr;
# вывести список доступных баз данных
\l;
# установка пароля для postgres
psql -U postgres -c "ALTER USER postgres PASSWORD 'add-root-password'"
# выход
\q
exit
```

Настраиваем порты
```
ufw allow 17312/tcp	# jira main port
ufw allow 17314/tcp	# confluence main port
ufw allow 17316/tcp	# jira control port
ufw allow 17318/tcp	# confluence control port
```

Вначале установим Jira, чтобы можно было цетрализованно работать с пользователями.

### Установка Jira 

```
# обновим пакеты
sudo apt update
sudo apt-get dist-upgrade
# скачиваем установочный пакет
mkdir /opt/jira/ && cd /opt/jira/
# стоит проверить версию здесь https://www.atlassian.com/ru/software/jira/download
sudo wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-8.11.0-x64.bin
# даем права на исполнение и устанавливаем
sudo chmod a+x atlassian-jira-software-8.11.0-x64.bin
sudo ./atlassian-jira-software-8.11.0-x64.bin
# в ходе установки указываем директории /home/atlassian/jira и /home/atlassian/application-data/jira, порты 17312 и 17316 и соглашаемся на установку системного демона (y)
# команды для управления
sudo service jira <start | stop | restart>
systemctl  <start | stop | status | restart> jira.service
# базовые директории для jira
/home/atlassian/jira
/home/atlassian/application-data/jira
# конфиг сервера
/home/atlassian/jira/conf/server.xmly
```

Далее переходим к настройке jira в браузере по соответствующим IP-адресу и порту, где указываем данные для подключения к БД и в качестве базового URL [https://jira.ln] можем указать домен, который создадим далее, а также данные нового пользователя. 

Создаем сертификат для выбранного URL:
```
cd /home/certs
./create_certificate_for_domain.sh jira.ln
```

Добавим домен и перезапустим DNS: 
```
# редактируем хосты
nano /etc/hosts
# добавляем запись с доменом
17.10.1.1       jira jira.ln
# сохраняем и перезапускаем DNS
systemctl restart dnsmasq
```

Создадим директории для логов и кэша:
```
mkdir /var/cache/nginx
mkdir /var/cache/nginx/jira
mkdir /var/log/nginx/jira
```

Настраиваем NGINX для проксирования на выбранный домен:
`nano /etc/nginx/sites-available/jira.conf`

Добавляем следующий конфиг
```nginx
proxy_cache_path /var/cache/nginx/jira levels=1:2 keys_zone=jira-cache:50m
max_size=50m inactive=1440m;

#SSL LISTENER
upstream atlassian-jira {
        server 17.10.1.1:17312 fail_timeout=0;
}

server {
        listen 443 ssl;
        listen [::]:443 ssl;

        ssl on;
        server_name jira.ln;
	set $server_url 'jira.ln';
	
        ssl_certificate     /home/certs/jira.ln/jira.ln.crt;
        ssl_certificate_key /home/certs/jira.ln/device.key;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/nginx/dhparam.pem;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';	
        add_header Strict-Transport-Security max-age=15768000;
        server_tokens off;

	# access_log off;
        access_log /var/log/nginx/jira/jira.access.log;		
        error_log /var/log/nginx/jira/jira.error.log;

        client_max_body_size 50M;

        ## Proxy settings
        proxy_max_temp_file_size    0;
        proxy_connect_timeout      900;
        proxy_send_timeout         900;
        proxy_read_timeout         900;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        proxy_intercept_errors     on;

        proxy_cache jira-cache;
        proxy_cache_key "$scheme://$host$request_uri";
        proxy_cache_min_uses 1;
        proxy_cache_valid 1440m;
        auth_basic off;
	
        location / {
                proxy_pass http://17.10.1.1:17312;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Forwarded-Server $host;

                set $do_not_cache 0;
                if ($request_uri ~* ^(/secure/admin|/plugins|/secure/projects|/projects|/admin)) {
                        set $do_not_cache 1;
                }
                proxy_cache_bypass $do_not_cache;
        }
}
```

Активируем конфиг и перезапускаем nginx:
```
ln -s /etc/nginx/sites-available/jira.conf /etc/nginx/sites-enabled/jira.conf
systemctl restart nginx
```

Далее настраиваем Jira:
`nano /home/atlassian/jira/conf/server.xml`
Необходимо закоментировать коннектор под http, а затем раскоментировать и исправить следующее:
```
        <Connector port="17312" relaxedPathChars="[]|" relaxedQueryChars="[]|{}^&#x5c;&#x60;&quot;&lt;&gt;"
                   maxThreads="150" minSpareThreads="25" connectionTimeout="20000" enableLookups="false"
                   maxHttpHeaderSize="8192" protocol="HTTP/1.1" useBodyEncodingForURI="true" redirectPort="8443"
                   acceptCount="100" disableUploadTimeout="true" bindOnInit="false" secure="true" scheme="https"
                   proxyName="jira.ln" proxyPort="443"/>
```

==================================================================================
?Далее требуется пробросить порты через iptables с 80 на 8080 и с 443 на 8443
```
/sbin/iptables -t nat -A OUTPUT -p tcp -d 127.0.0.1,127.0.0.1 --dport 443 -j  REDIRECT --to-port 8443
```

Проверим порты при помощи:
```
lsof -i :80
lsof -i :8080
lsof -i :8443
lsof -i :443
```
=====================================================================================

Так как мы используем собственные сертификаты для SSl/HTTPS, нам необходимо добавить самоподписанный сертификат для выбранного домена в keystore для java tomcat. Для этого нужно преобразовать ранее выпущенный сертификат и затем его добавить. При преобразовании сертификата нас попросят ввести пароль, который мы используем в следующей команде при добавлении после ключа -srcstorepass - нужно заменить src-changeit на введенный пароль при конвертировании ключа. По умолчанию jira keystore имеет пароль changeit.

```
openssl pkcs12 -export -in jira.ln/jira.ln.crt -inkey jira.ln/device.key \
               -out servkeystore.p12 -name ca \
               -CAfile rootCA.crt -caname root

keytool -importkeystore -srckeystore /home/certs/servkeystore.p12 \
			-destkeystore /home/atlassian/jira/jre/lib/security/cacerts 
			-deststoretype pkcs12 -alias ca -deststorepass changeit -srcstorepass src-changeit -validity 3650
```

Посмотреть пользователя, который является системным для Jira можно так:
```
nano /home/atlassian/jira/bin/user.sh
```

Если использовался не самоподписанный сертификат, однако при открытии Dashboard появляется ошибка 500 при загрузке Jira Gadgets, следует проверить BASE_URL в настройках Jira в панели администрирования, и если всё в порядке, то следует добавить полученный сертификат в keystore.


### Установка Confluence

```
# скачиваем установочный пакет
mkdir /opt/confluence/ && cd /opt/confluence/
# стоит проверить версию здесь https://www.atlassian.com/ru/software/confluence/download
sudo wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-7.6.1-x64.bin
# даем права на исполнение и устанавливаем
sudo chmod a+x atlassian-confluence-7.6.1-x64.bin
sudo ./atlassian-confluence-7.6.1-x64.bin
# в ходе установки указываем директории /home/atlassian/confluence и /home/atlassian/application-data/confluence, порты 17314 и 17318 и соглашаемся на установку системного демона (y)
# на вопрос Start Confluence now? отвечаем n
# перед первым запуском изменим настройки
# откроем требуемые порты 
ufw allow 17314/tcp
ufw allow 17318/tcp

# команды для управления
sudo service confluence <start | stop | restart>
systemctl  <start | stop | status | restart> confluence.service
# базовые директории для Confluence
/home/atlassian/confluence 
/home/atlassian/application-data/confluence
# конфиг сервера
/home/atlassian/confluence/conf/server.xml

# при необходимости для полного удаления выполнить следующее
systemctl  stop confluence.service
cd /home/atlassian/confluence/
sudo chmod a+x uninstall
./uninstall
```

Далее переходим к настройке Confluence в браузере по соответствующим IP-адресу и порту, где указываем данные для подключения к БД и в качестве базового URL [https://confluence.ln] можем указать домен, который создадим далее, а также данные нового пользователя. Если при интеграции с Jira появляется ошибка "Отказано в доступе", то как правило проблема в самоподписанном сертификате, который нужно добавить в keystore confluence, так же как ранее это было сделано для Jira. При этом интеграцию стоит произвести после настройки Сonfluence, указав данные пользователя-администратора из Jira. Интагрцию производим потом при помощи Application Links в Jira, указав взаимную ссылку в Confluence (Раздел настроек: "Связи с приложениями").

Создаем сертификат для выбранного URL:
```
cd /home/certs
./create_certificate_for_domain.sh confluence.ln
```

Отметим также, что был взят домен confluence.ln потому, как домен wiki.ln используем для Wikijs.
Добавим домен и перезапустим DNS: 
```
# редактируем хосты
nano /etc/hosts
# добавляем запись с доменом
17.10.1.1       confluence confluence.ln
# сохраняем и перезапускаем DNS
systemctl restart dnsmasq
```


Так как мы используем собственные сертификаты для SSl/HTTPS, нам необходимо добавить самоподписанный сертификат для выбранного домена в keystore для java tomcat. Для этого нужно преобразовать ранее выпущенный сертификат и затем его добавить. При преобразовании сертификата нас попросят ввести пароль, который мы используем в следующей команде при добавлении после ключа -srcstorepass - нужно заменить src-changeit на введенный пароль при конвертировании ключа. По умолчанию confluence keystore имеет пароль changeit.

```
openssl pkcs12 -export -in jira.ln/jira.ln.crt -inkey jira.ln/device.key -out jservkeystore.p12 -name jira -CAfile rootCA.crt -caname root


# сертификаты для confluence
keytool -importkeystore -srckeystore /home/certs/jservkeystore.p12 -destkeystore /home/atlassian/confluence/jre/lib/security/cacerts -deststoretype pkcs12 -alias jira -deststorepass changeit -srcstorepass Lz5kSHc0WOs% -validity 3650
keytool -importkeystore -srckeystore /home/certs/servkeystore.p12 -destkeystore /home/atlassian/confluence/jre/lib/security/cacerts -deststoretype pkcs12 -alias ca -deststorepass changeit -srcstorepass Lz5kSHc0WOs% -validity 3650

# сертификаты для jira
keytool -importkeystore -srckeystore /home/certs/jservkeystore.p12 -destkeystore /home/atlassian/jira/jre/lib/security/cacerts -deststoretype pkcs12 -alias jira -deststorepass changeit -srcstorepass Lz5kSHc0WOs% -validity 3650
keytool -importkeystore -srckeystore /home/certs/servkeystore.p12 -destkeystore /home/atlassian/jira/jre/lib/security/cacerts -deststoretype pkcs12 -alias ca -deststorepass changeit -srcstorepass Lz5kSHc0WOs% -validity 3650

```


Создадим директории для логов и кэша:
```
mkdir /var/cache/nginx/confluence
mkdir /var/log/nginx/confluence
```

Настраиваем NGINX для проксирования на выбранный домен:
`nano /etc/nginx/sites-available/confluence.conf`

Добавляем следующий конфиг
```nginx
upstream atlassian-confluence {
        server 17.10.1.1:17314 fail_timeout=0;
}

server {
        listen 443 ssl;
        listen [::]:443 ssl;

        ssl on;
        server_name confluence.ln;
	set $server_url 'confluence.ln';
	
        ssl_certificate     /home/certs/confluence.ln/confluence.ln.crt;
        ssl_certificate_key /home/certs/confluence.ln/device.key;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/nginx/dhparam.pem;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';	
        add_header Strict-Transport-Security max-age=15768000;
        server_tokens off;

	access_log off;		
        error_log /var/log/nginx/confluence/confluence.error.log;

        client_max_body_size 500M;

        ## Proxy settings
        proxy_max_temp_file_size    0;
        proxy_connect_timeout      900;
        proxy_send_timeout         900;
        proxy_read_timeout         900;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        proxy_intercept_errors     on;
        auth_basic off;
	
        location / {
                proxy_pass http://17.10.1.1:17314;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Forwarded-Server $host;
		
        }
}
```

Активируем конфиг и перезапускаем nginx:
```
ln -s /etc/nginx/sites-available/confluence.conf /etc/nginx/sites-enabled/confluence.conf
systemctl restart nginx
```

Далее настраиваем Confluence:
`nano /home/atlassian/confluence/conf/server.xml`
Необходимо закоментировать коннектор под http, а затем раскоментировать и исправить следующее:
```xml
        <Connector port="17314" connectionTimeout="20000" redirectPort="8443"
                   maxThreads="48" minSpareThreads="10"
                   enableLookups="false" acceptCount="10" debug="0" URIEncoding="UTF-8"
                   protocol="org.apache.coyote.http11.Http11NioProtocol"
                   scheme="https" secure="true" proxyName="confluence.ln" proxyPort="443"/>
```

==================================================================================
?Далее требуется пробросить порты через iptables с 80 на 8080 и с 443 на 8443
```
/sbin/iptables -t nat -A OUTPUT -p tcp -d 127.0.0.1,127.0.0.1 --dport 443 -j  REDIRECT --to-port 8443
```

Проверим порты при помощи:
```
lsof -i :80
lsof -i :8080
lsof -i :8443
lsof -i :443
```
=====================================================================================


Посмотреть пользователя, который является системным для Confluence можно так:
```
nano /home/atlassian/confluence/bin/user.sh
```

Если использовался не самоподписанный сертификат, однако при открытии Dashboard появляется ошибка 500 при загрузке Jira Gadgets, следует проверить BASE_URL в настройках Jira в панели администрирования, и если всё в порядке, то следует добавить полученный сертификат в keystore.

Стоит также установить плагин Draw.io для создания диаграмм в Confluence. В управлении приложениями можно включить стандартные макросы HTML (по умолчанию они отключены). Не забываем их отключить при установке на production сервер.

## Установка GitLab

```
# установить требуемые зависимости
sudo apt-get update
# при установке postfix выбираем "Internet Site" и вводим домен для почты, например, mail.ln
sudo apt-get install -y curl openssh-server ca-certificates postfix

# добавить репозиторий
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
# установить gitlab-ce
sudo apt-get install gitlab-ce

# добавим домен
nano /etc/hosts
# добавляем
17.10.1.1 gitlab gitlab.ln
# перезапустим dns
service dnsmasq restart

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

Учитываем, что для работы GitLab Puma потребуется порт 8080, поэтому изменим данный порт на 19000 в конфиге следующим образом:
```
### Advanced settings
puma['listen'] = '127.0.0.1'
puma['port'] = 19000
puma['socket'] = '/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket'
puma['pidfile'] = '/opt/gitlab/var/puma/puma.pid'
puma['state_path'] = '/opt/gitlab/var/puma/puma.state'
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
# выходим
\q
exit
```
gitlab-ctl pg-password-md5 gitlab

Затем нужно добавить в /etc/gitlab/gitlab.rb следующее:
```
# откроем в редакторе
sudo nano /etc/gitlab/gitlab.rb

# добавим настройки базы данных
gitlab_rails['db_host'] = '127.0.0.1'
gitlab_rails['db_port'] = 5432
gitlab_rails['db_username'] = "gitlabusr"
gitlab_rails['db_password'] = "bFET9BD7!Wwc"

# force ssl on all connections defined in trust_auth_cidr_addresses and md5_auth_cidr_addresses
postgresql['hostssl'] = false
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
gitlab-ctl tail
```

При первой загрузке страницы GitLab будет предложено ввести новый пароль для пользователя root, при помощи которого можно получить доступ к Admin Area. Если есть проблемы с доступом через localhost, то стоит его добавить в Admin Area -> Settings -> Network -> Outbound requests -> установить галочку Allow requests to the local network from web hooks and services и в поле ниже ввести хост сервера (127.0.0.1 и т.п.), можно также указывать конкретный порт.

Выпустим сертификат:
```
cd /home/certs
./create_certificate_for_domain.sh gitlab.ln
```

Теперь закончим настроку NGINX, добавив следующий конфиг:
`nano /etc/nginx/sites-available/gitlab.conf`

```nginx
upstream gitlab-workhorse {
	server unix:/var/opt/gitlab/gitlab-workhorse/socket;
}

server {
	listen 80;
	listen [::]:80;
	server_name gitlab.ln;
	return 302 https://$host$request_uri;
}

server {
	listen 443 ssl;
	listen [::]:443 ssl;
	ssl on;
	server_name gitlab.ln;
    
	ssl_certificate     /home/certs/gitlab.ln/gitlab.ln.crt;
    ssl_certificate_key /home/certs/gitlab.ln/device.key;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/nginx/dhparam.pem;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';	
    add_header Strict-Transport-Security max-age=15768000;
    server_tokens off;
	
	proxy_set_header X-Forwarded-For $remote_addr;
    client_max_body_size 250m;
	
	root /opt/gitlab/embedded/service/gitlab-rails/public;

	access_log  /var/log/gitlab/nginx/gitlab_access.log;
    error_log   /var/log/gitlab/nginx/gitlab_error.log;
        
    location / {
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Forwarded-Server $host;
                proxy_pass http://gitlab-workhorse;
    }

	location ~ ^/[\w\.-]+/[\w\.-]+/(info/refs|git-upload-pack|git-receive-pack)$ {
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}
	location ~ ^/[\w\.-]+/[\w\.-]+/repository/archive {
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}
	location ~ ^/api/v3/projects/.*/repository/archive {
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}
	# Build artifacts should be submitted to this location
	location ~ ^/[\w\.-]+/[\w\.-]+/builds/download {
		client_max_body_size 0;
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}
	# Build artifacts should be submitted to this location
	location ~ /ci/api/v1/builds/[0-9]+/artifacts {
		client_max_body_size 0;
		# 'Error' 418 is a hack to re-use the @gitlab-workhorse block
		error_page 418 = @gitlab-workhorse;
		return 418;
	}

	location @gitlab-workhorse {
		## https://github.com/gitlabhq/gitlabhq/issues/694
		## Some requests take more than 30 seconds.
		proxy_read_timeout      3600;
		proxy_connect_timeout   300;
		proxy_redirect          off;
		# Do not buffer Git HTTP responses
		proxy_buffering off;
		proxy_set_header    Host                $http_host;
		proxy_set_header    X-Real-IP           $remote_addr;
		proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
		proxy_set_header    X-Forwarded-Proto   $scheme;
		proxy_pass http://gitlab-workhorse;
		## Pass chunked request bodies to gitlab-workhorse as-is
		proxy_request_buffering off;
		proxy_http_version 1.1;
	}

	error_page 502 /502.html;

}
```

Активируем конфиг и перезапустим NGINX
```
ln -s /etc/nginx/sites-available/gitlab.conf /etc/nginx/sites-enabled/gitlab.conf
service nginx restart
```

Итоговая конфигурация GitLab:
`nano /etc/gitlab/gitlab.rb`

```ruby
# основные настройки GitLab
external_url 'http://gitlab.ln'
nginx['enable'] = false
web_server['external_users'] = ['www-data']
gitlab_rails['trusted_proxies'] = [ '127.0.0.1', '17.10.1.1' ]
gitlab_workhorse['listen_network'] = "unix"
gitlab_workhorse['listen_addr'] = "/var/opt/gitlab/gitlab-workhorse/socket"
# настройки базы данных PostgreSQL
postgresql['enable'] = false
gitlab_rails['db_adapter'] = "postgresql"
gitlab_rails['db_host'] = '127.0.0.1'
gitlab_rails['db_port'] = 5432
gitlab_rails['db_username'] = "gitlabusr"
gitlab_rails['db_password'] = "bFET9BD7!Wwc"
# настройки кэша
redis['enable'] = false
gitlab_rails['redis_host'] = '127.0.0.1'
gitlab_rails['redis_port'] = 6379
gitlab_rails['redis_password'] = 'bxP5xk%1LRx3'
### переопределение порта puma
puma['listen'] = '127.0.0.1'
puma['port'] = 19000
puma['socket'] = '/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket'
puma['pidfile'] = '/opt/gitlab/var/puma/puma.pid'
puma['state_path'] = '/opt/gitlab/var/puma/puma.state'
```


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

mysqladmin -u root password <root-password> 

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
         proxy_pass         http://17.10.1.1:17101;
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
         proxy_pass http://17.10.1.1:17100;
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
SERVICE_URL = http://17.10.1.1:17101

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
bind = "17.10.1.1:17101"

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
default = 100

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
SITE_BASE                           = 'http://17.10.1.1'
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

FILE_SERVER_ROOT                    = 'http://17.10.1.1/seafhttp'
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

Dark theme: https://github.com/u-alexey/seafile_ce-edition_custom_theme

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
Seafile Settings: https://seafile.gitbook.io/seafile-server-manual/server-configuration-and-customization/seahub_settings.py

## Run on https (use SSL cert)

Когда мы выпустили сертификат под выбранный домен, нам необходимо изменить настройки в NGINX следующим образом:
```nginx
log_format seafileformat '$http_x_forwarded_for $remote_addr [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $upstream_response_time';

server {
        listen       80;
        server_name  mars.sol;
        rewrite ^ https://$http_host$request_uri? permanent;
        server_tokens off;
}

server {
        listen 443;
        ssl on;

        server_name mars.sol;

        ssl_certificate /home/certs/mars.sol.crt;
        ssl_certificate_key /home/certs/device.key;

        ssl_session_timeout 5m;
        ssl_session_cache shared:SSL:5m;

        # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
        ssl_dhparam /etc/nginx/dhparam.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-CAMELLIA256-SHA:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-SEED-SHA:DHE-RSA-CAMELLIA128-SHA:HIGH:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!SRP:!DSS';
        ssl_prefer_server_ciphers on;
	
        proxy_set_header X-Forwarded-For $remote_addr;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
        server_tokens off;
	
        location / {
                proxy_pass         http://17.10.1.1:17101;
                proxy_set_header   Host $host:$proxy_port;
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Host $server_name;
                proxy_set_header   X-Forwarded-Proto https;
                proxy_read_timeout  1200s;
                client_max_body_size 0;

                access_log      /var/log/nginx/seahub.access.log seafileformat;
                error_log       /var/log/nginx/seahub.error.log;
        }
        location /seafhttp {
                rewrite ^/seafhttp(.*)$ $1 break;
                proxy_pass http://17.10.1.1:17100;
                client_max_body_size 0;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_connect_timeout  36000s;
                proxy_read_timeout  36000s;
                proxy_send_timeout  36000s;
                send_timeout  36000s;
                proxy_request_buffering off;
        }

        location /media {
                root /home/seafile/seafile-server-latest/seahub;
        }

        location /seafdav {
                proxy_pass         http://17.10.1.1:8080/seafdav;
                proxy_set_header   Host $host:$proxy_port;
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Host $server_name;
                proxy_set_header   X-Forwarded-Proto https;
                proxy_read_timeout  1200s;

                client_max_body_size 0;
                proxy_request_buffering off;
        }
}
```

Далее перезагружаем NGINX:
```
systemctl restart nginx
```

Подробная инструкция: https://seafile.gitbook.io/seafile-server-manual/deploying-seafile-under-linux/enabling-https-with-nginx

## Seafile CLI клиент для терминала Debian

Подробная инструкция: https://download.seafile.com/published/seafile-user-manual/syncing_client/linux-cli.md


# Установка JupyterLab

Добавим отдельного пользователя:
```
# создадим нового пользователя
useradd --system --comment "Jupyter Lb" jupyter --home-dir  /home/jupyter -m -U -s /bin/false
# зададим пароль пользователя
passwd jupyter
# добавим пользователя в группу nginx
usermod -a -G jupyter www-data
```

Используем менеджер pip:
```
# устанавливаем и обновляем jupyterlab
pip3 install jupyterlab
pip3 install jupyterlab --upgrade
# устанавливаем дополнительные пакеты
pip3 install --trusted-host pypi.org --trusted-host files.pythonhosted.org pandas
# запускаем jupyter lab
jupyter lab
# создадим отдельного пользователя

# добавляем в systemd для автозапуска
sudo nano /etc/systemd/system/jupyter.service

# =============================================================
# Прописываем следующее:
# =============================================================
[Unit]
Description=Jupyter Lab     
After=syslog.target network.target

[Service]
Type=oneshot    # or forking / simple
ExecStart=/usr/local/bin/jupyter lab --ip 17.10.1.1 --port 9090
ExecStop=/usr/local/bin/jupyter lab stop
WorkingDirectory=/home/jupyter/
User=jupyter
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
chown -R jupyter:jupyter /home/jupyter

# restart 
systemctl restart jupyter


# получить токен от созданного пользователя можно будет так
sudo -u jupyter jupyter notebook list

# generate config and add password
jupyter lab --generate-config	# like user
jupyter notebook password 	# like root
```

Конфигурация NGINX для SSL (не забываем выпустить сертификат):
`nano /etc/nginx/sites-available/jupyter.conf`

```nginx
server {
        listen 80;
        server_name jupyter.ln;
        return 302 https://$host$request_uri;
}

server {
        listen 443;
        ssl on;
        server_name jupyter.ln;

        ssl_certificate /home/certs/jupyter.ln/jupyter.ln.crt;
        ssl_certificate_key /home/certs/jupyter.ln/device.key;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/nginx/dhparam.pem;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
        ssl_session_timeout 5m;
        ssl_stapling on;
        ssl_stapling_verify on;
	
        add_header Strict-Transport-Security max-age=15768000;
        server_tokens off;
	
        location / {
                proxy_pass http://17.10.1.1:9090;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Host $server_name;
                proxy_set_header   X-Forwarded-Proto https;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
        }

        client_max_body_size 50M;
        error_log /var/log/nginx/error.log;

        location ~* /(api/kernels/[^/]+/(channels|iopub|shell|stdin)|terminals/websocket)/? {
                proxy_pass http://17.10.1.1:9090;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
                proxy_set_header Upgrade "websocket";
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
        }
}
```

Активируем конфиг и перезагрузим nginx:
```
ln -s /etc/nginx/sites-available/jupyter.conf /etc/nginx/sites-enabled/jupyter.conf
systemctl restart nginx
```






## DynDNS

https://account.dyn.com/

