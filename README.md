# Minimal Dev Lab on Linux (local network dev host)

## Первоначальная настройка

### Create user and add his to sudo group

```
# повышаем права до root и переходим в домашнюю директорию root-пользователя
sudo su
cd
# устанавливаем sudo (если отсутствует)
apt-get install sudo
visudo	# файл с настройками
# Создаем пользователя (указываем пароль и др. данные) и добавляем его в sudo
sudo adduser <username>
sudo usermod -aG sudo <username>
```

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

# deb http://deb.debian.org/debian/ buster-backports main contrib non-free
# deb-src http://deb.debian.org/debian/ buster-backports main contrib non-free

# сохраняем изменения: Ctrl+X затем Y и Enter
# обновим зависимости
sudo apt update
sudo apt -y dist-upgrade
```

## Установим временную зону (дату и время в своём регионе)
 
Checking the Current Timezone:
```
timedatectl
```
The system timezone is configured by symlinking /etc/localtime to a binary timezone identifier in the /usr/share/zoneinfo directory. Other option to check the timezone is to show the path the symlink points to using the ls command :
```
ls -l /etc/localtime
```
Changing Timezone
To list all available time zones, you can either list the files in the /usr/share/zoneinfo directory or use the timedatectl command.
```
timedatectl list-timezones
```
Once you identify which time zone is accurate to your location, run the following command as sudo user:
```
sudo timedatectl set-timezone your_time_zone
```
For example:
```
sudo timedatectl set-timezone Europe/Moscow
```
Verify the change by issuing the timedatectl command:
```
timedatectl
```

### Установим базовые компоненты

```bash
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
	certbot python3-certbot-nginx \
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
sudo apt-get update && sudo apt-get install -y yarn
sudo apt-get install -y nodejs
# обновляем зависимости и пакеты
apt update

# включаем FireWall
# просмотрим установленные правила firewall 
ufw enable && ufw status

# устанавливаем дополнительные пакеты pip
sudo apt-get install -y python3-pip python-pip python3-dev python-dev

# you can upgrade pip version: pip install --user --upgrade pip or pip3 install --upgrade --user pip

python3 -m pip install virtualenv
python3 -m pip install virtualenvwrapper

# очистим старые записи после обновления -> hash -d pip

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
cat /etc/os-release && echo -n "arch: " && arch && git --version && python3 -V && pip3 -V && echo -n "nodejs: " && nodejs -v && echo -n "npm: " && npm -v
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

dns-nameservers 17.10.1.1

# перезапускаем сеть
systemctl restart networking
# проверяем настройки сети
ip a
# если интерфейс не поднялся, то выполняем
sudo ifup enp3s2
```

## Установка DHCP и DNS сервера 

Установка и настройка DHCP сервера [источник]: https://www.server-world.info/en/note?os=Debian_10&p=dhcp&f=1

```
# устанавливаем DHCP сервер
apt -y install isc-dhcp-server
# настроим конфиг только для IPv4

# ------------------------------------------------------------------------
# Добавляем следующее в /etc/default/isc-dhcp-server
# ------------------------------------------------------------------------
# строка 4: раскомментируем
DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
# строки 17,18: указываем интерфейсы для прослушивания
# если не используем IPv6, то закомментируем строку
INTERFACESv4="enp3s2"
# INTERFACESv6=""
# ------------------------------------------------------------------------

# ------------------------------------------------------------------------
# Добавляем следующее в /etc/dhcp/dhcpd.conf
# ------------------------------------------------------------------------
# comment out this lines
# option domain-name ".ln";
# option domain-name-servers ns1.example.org, ns2.example.org;

# add this
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
authoritative;

subnet 17.10.1.0 netmask 255.255.255.0 {
    # specify default gateway
    option routers      17.10.1.1;
    # specify subnet-mask
    option subnet-mask  255.255.255.0;
    option domain-name-servers 17.10.1.1;
    # specify the range of leased IP address
    range dynamic-bootp 17.10.1.2 17.10.1.254;
}
# ------------------------------------------------------------------------

# restart DHCP server
systemctl restart isc-dhcp-server 
```


Use DNSmasq as DNS server:
```
sudo apt -y install dnsmasq resolvconf

# Разрешение трафика на локальном узле
sudo iptables -A INPUT -i enp3s2 -j ACCEPT
sudo /sbin/iptables-save	# сохраняем правила
apt-get install iptables-persistent	# установим для автоподгрузки файлов после преезапуска системы
sudo ufw allow bootps	

# if needed:
# iptables -I INPUT -p tcp --dport 53 -j ACCEPT
# iptables -I INPUT -p udp --dport 53 -j ACCEPT
# iptables -I INPUT -p tcp --dport 5353 -j ACCEPT
# iptables -I INPUT -p udp --dport 5353 -j ACCEPT	
```

Manual: https://www.server-world.info/en/note?os=Debian_10&p=dnsmasq&f=1
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
# edit hosts-file
nano /etc/hosts
# add this
17.10.1.1 cloud cloud.ln
# Если возникнут проблемы, проверьте конфигурацию брандмауэра! 

# Конфигурация dnsmasq выполняется в файле /etc/dnsmasq.conf
cp /etc/dnsmasq.conf /etc/dnsmasq.conf.sample
nano /etc/dnsmasq.conf

# ------------------------
# add this configs
# ------------------------
# domain-needed
# bogus-priv
# strict-order

interface=enp3s2

# DNS config
local=/ln/
domain=ln
expand-hosts
# ------------------------

systemctl restart dnsmasq
systemctl restart ifup@enp3s2 resolvconf
systemctl restart networking


# edit /etc/dhcp/dhclient.conf
nano /etc/dhcp/dhclient.conf
# uncomment this string
prepend domain-name-servers 127.0.0.1;

# restart dnsmasq and networks
systemctl restart networking
/etc/init.d/dnsmasq restart
```

More info: https://modx.cc/linux/programma-dnsmasq-(dhcp-i-server-imen)/

Now, if we on client open terminal/cmd and write `ping cloud.ln`, we get request from 17.10.1.1. If we use some web-servers like nginx, we can open in browser url: http://cloud.ln [17.10.1.1].

```
# check in cmd
ping cloud.ln
nslookup cloud.ln
```

We can edit our /etc/hosts on server and add domains for our ip's.


### Доступ по SSH

Обнович ssh на windows для подключения с последней версией протокола:

Manual here: https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH

Если мы хотим получать доступ к серверу по SSH, то стоит отключить возможность входа от суперпользователя и доступ по паролю.

Создаем ssh-ключ для пользователя (not root user)
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
./create_certificate_for_domain.sh cloud.ln
```

Получаем два файла: cloud.ln и device.key в директории /home/certs/cloud.ln

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


Добавить сертификат в доверенные на Debian можно так:
```
apt-get install ca-certificates
cp cacert.pem /usr/share/ca-certificates
dpkg-reconfigure ca-certificates
```

Подробнее про добавление сертификата в доверенные в Linux: https://unix.stackexchange.com/questions/90450/adding-a-self-signed-certificate-to-the-trusted-list

Добавить утилиту nslookup можно так:
`apt-get install -y dnsutils`

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
sudo apt-get install -y aptitude
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

### Установка PostgreSQL 12.x

```bash
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL
sudo apt-get install -y postgresql-12

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
useradd --system --comment "WikiJS User" wikiuser --home-dir  /home/wiki
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

Для запуска через NGINX мы должны проксировать запросы с домена, привязанного к ip сервера на локальный хост и указанный при установке порт 127.0.0.1:3000
Для этого создадим конфиг в NGINX:
`nano /etc/nginx/sites-available/wikijs.conf`

```nginx
server {
        listen 80;
        listen [::]:80;
        server_name know.lnovus.com;
        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

#        ssl on;
        server_name know.lnovus.com;
#        set $server_url 'lnovus.com';

        ssl_certificate     /etc/letsencrypt/live/lnovus.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/lnovus.com/privkey.pem;

        charset utf-8;
        client_max_body_size 0;

        location / {
                proxy_pass http://127.0.0.1:3000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Forwarded-Server $host;
                proxy_set_header   X-Forwarded-Proto https;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
                proxy_next_upstream error timeout http_502 http_503 http_504;

                if ($host !~ ^(know.lnovus.com|www.know.lnovus.com)$ ) {
                        return 444;
                }
                if ($http_user_agent ~* LWP::Simple|BBBike|wget) {
                        return 403;
                }
                if ($http_user_agent ~* msnbot|scrapbot) {
                        return 403;
                }
        }
}

```

Активируем конфиг и перезапустим NGINX:
```
ln -s /etc/nginx/sites-available/wikijs.conf /etc/nginx/sites-enabled/wikijs.conf
service nginx restart
```

### Установим дополнительные сервисы

Выбираем последнюю версию на https://github.com/jgm/pandoc/releases/tag/2.10.1

```bash
# скачиваем deb пакет
wget https://github.com/jgm/pandoc/releases/download/2.10.1/pandoc-2.10.1-1-amd64.deb
# устанавливаем
sudo dpkg -i pandoc-2.10.1-1-amd64.deb
# проверяем
pandoc -v
# устанавливаем sharp - использовать стандартного пользователя (не root)
npm install sharp

# перезапускаем и проверяем
service wiki restart
service nginx restart
```


# Установка Jira & Confluence

Создание базы данных в postgres:
```
su - postgres
psql
# создание базы данных под confluence и пользователя для atlassian продуктов
CREATE DATABASE confluencedb with encoding='UNICODE';
CREATE USER atlassianusr with password 'add-user-password';
GRANT ALL PRIVILEGES ON DATABASE confluencedb TO atlassianusr;
# создание базы данных под jira
CREATE DATABASE jiradb with encoding='UNICODE';
GRANT ALL PRIVILEGES ON DATABASE jiradb TO atlassianusr;
# вывести список доступных баз данных
\l
# установка пароля для postgres
ALTER USER postgres PASSWORD 'add-root-password'
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

## Установка Jira 

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
/home/atlassian/jira/conf/server.xml
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
proxy_cache_path /var/cache/nginx/jira levels=1:2 keys_zone=jira-cache:10m 
max_size=500m inactive=1440m;

upstream atlassian-jira {
	server 17.10.1.1:17312 fail_timeout=0;
}

server {
        listen 80;
        listen [::]:80;
        server_name jira.ln;
        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
	listen [::]:443 ssl http2;

	ssl on;
        server_name jira.ln;	
	set $server_url 'jira.ln';

        ssl_certificate     /home/certs/jira.ln/jira.ln.crt;
        ssl_certificate_key /home/certs/jira.ln/device.key;

        client_max_body_size 0;

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
		proxy_pass http://127.0.0.1:17312;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;		
		proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Forwarded-Server $host;

		## Proxy settings
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
openssl pkcs12 -export -in jira.ln/jira.ln.crt -inkey jira.ln/device.key -out servkeystore.p12 -name ca -CAfile rootCA.crt -caname root

keytool -importkeystore -srckeystore /home/certs/servkeystore.p12 -destkeystore /home/atlassian/jira/jre/lib/security/cacerts -deststoretype pkcs12 -alias ca -deststorepass changeit -srcstorepass 7w0*8wxHGvcC -validity 3650
```

Посмотреть пользователя, который является системным для Jira можно так:
```
nano /home/atlassian/jira/bin/user.sh
```

Если использовался не самоподписанный сертификат, однако при открытии Dashboard появляется ошибка 500 при загрузке Jira Gadgets, следует проверить BASE_URL в настройках Jira в панели администрирования, и если всё в порядке, то следует добавить полученный сертификат в keystore.

Если при перезагрузке системы Jira не запускается автоматически, то смотрим инструкцию:
https://confluence.atlassian.com/jirakb/start-jira-applications-automatically-in-linux-828796713.html
https://community.atlassian.com/t5/Jira-Software-questions/Jira-autostart-problem-on-Debian-9/qaq-p/624909

## Устранение проблем с автозапуском Jira

По умолчанию сервис Jira будет прописан в /etc/init.d/jira, но мы создадим сервис в system.d, а данный сервис на всякий случай перенесем в /opt (в качестве backup):
```
# перенесем сервис по умолчанию - бэкап
mv /etc/init.d/jira /etc/init.d/jira.old
# создадим свой сервис
nano /lib/systemd/system/jira.service
# добавим следующее
[Unit]
Description=Atlassian Jira
After=network.target

[Service]
Type=forking
User=jira
PIDFile=/home/atlassian/jira/work/catalina.pid
ExecStart=/home/atlassian/jira/bin/start-jira.sh
ExecStop=/home/atlassian/jira/bin/stop-jira.sh

[Install]
WantedBy=multi-user.target

# сохраним и активируем
systemctl enable jira.service
systemctl start jira.service
# посмотрим статус сервиса
systemctl status jira.service
# перезагрузим сервер и проверим статус снова
```


## Установка Confluence

```
# скачиваем установочный пакет
mkdir /opt/confluence/ && cd /opt/confluence/
# стоит выбирать версию, исходя из ранее установленной, чтобы восстановить данные из сделанных бэкапов
# можно проверить последнюю версию здесь https://www.atlassian.com/ru/software/confluence/download
sudo wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-7.6.2-x64.bin
# даем права на исполнение и устанавливаем
sudo chmod a+x atlassian-confluence-7.6.2-x64.bin
sudo ./atlassian-confluence-7.6.2-x64.bin
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
# create and add confluence key to confluence keystore

openssl pkcs12 -export -in confluence.ln/confluence.ln.crt -inkey confluence.ln/device.key -out confluenceservkeystore.p12 -name confluence -CAfile rootCA.crt -caname confluenceroot

keytool -importkeystore -srckeystore /home/certs/confluenceservkeystore.p12 -destkeystore /home/atlassian/confluence/jre/lib/security/cacerts -deststoretype pkcs12 -alias confluence -deststorepass changeit -srcstorepass 7w0*8wxHGvcC -validity 3650

# add cofnluence key to jira keystore

keytool -importkeystore -srckeystore /home/certs/confluenceservkeystore.p12 -destkeystore /home/atlassian/jira/jre/lib/security/cacerts -deststoretype pkcs12 -alias confluence -deststorepass changeit -srcstorepass 7w0*8wxHGvcC -validity 3650

# create and add jira key to jira keystore 

openssl pkcs12 -export -in jira.ln/jira.ln.crt -inkey jira.ln/device.key -out jservkeystore.p12 -name jira -CAfile rootCA.crt -caname root

keytool -importkeystore -srckeystore /home/certs/jservkeystore.p12 -destkeystore /home/atlassian/jira/jre/lib/security/cacerts -deststoretype pkcs12 -alias jira -deststorepass changeit -srcstorepass 7w0*8wxHGvcC -validity 3650

# add jira key to confluence keystore
keytool -importkeystore -srckeystore /home/certs/jservkeystore.p12 -destkeystore /home/atlassian/confluence/jre/lib/security/cacerts -deststoretype pkcs12 -alias jira -deststorepass changeit -srcstorepass 7w0*8wxHGvcC -validity 3650

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
        listen 80;
        listen [::]:80;
        server_name confluence.ln;
        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl on;
        server_name confluence.ln;
	set $server_url 'confluence.ln';
	
        ssl_certificate     /home/certs/confluence.ln/confluence.ln.crt;
        ssl_certificate_key /home/certs/confluence.ln/device.key;

        client_max_body_size 0;

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

Домашняя директория Home Confluence расположится в /hpme/atlassian-application-data/confluence

## Создание бэкапов и восстановление

### Бэкапы и восстановление Jira

Как правило, файлы бэкапов создаются в директории <backups-drive-dir>/atlassian/application-data/jira/export. Бэкапы представляют собой xml файлы и attachments в архивах типа zip с именованием по соответствующим датам. Необходимо перенести выбранных архив в директорию /home/atlassian/application-data/jira/import/ и установить соответствующие права:

```
cp 2020-Jul-29--1900.zip  /home/atlassian/application-data/jira/import/
chown jira:jira /home/atlassian/application-data/jira/import/2020-Jul-29--1900.zip
chmod 777 /home/atlassian/application-data/jira/import/2020-Jul-29--1900.zip
```

Далее, открываем Jira в браузере и переходим в Administration -> System -> Import & Export -> Restore System и указываем имя архива, например, 2020-Jul-29--1900.zip, а затем Restore и ждем завершения операции. 

Стоит учитывать, что прикрепленные файлы для некоторых задач могут не открыться, а также, что может появиться ошибка в Application links, т.к. возможно, Confluence или др. сервис, еще не был установлен, или подключен к данному экземпляру Jira. 
Стоит также выполнить Full Re-Index.

### Бэкапы и восстановление Confluence

Для Production рекомендуется отдельно создавать бэкапы базы данных и бэкапы домашней директории Confluence. В данном случае, изначально делаем бэкап базы данных Postgres:
```
su - postgres
# бэкап отдельной базы 
pg_dump dbname > dbname.bak
# бэкап всех баз данных PostreSQL
pg_dumpall > /home/postgres_backups/pg_dump_all.sql > /home/postgres_backups/pg_dump.log
# добавить задание в крон можно так
# откроем список заданий для пользователя postgres
crontab -e -u postgres
# добавим задание по бэкапам на 00:30 ночи на каждый день
# для всех баз данных
30 0 * * * pg_dumpall > /home/postgres_backups/pg_dump_all.sql > /home/postgres_backups/pg_dump.log
# для отдельной базы confluence
30 0 * * * pg_dump dbname > dbname.bak

# восстанавливать базу данных можно следующим образом
# изначально установим confluence 
# после процедуры настройки системы в браузере удалим базу данных "по умолчанию"
dropdb confluencedb
# создадим новую пустую базу данных
createdb confluencedb
# восстановим базу данных из бэкапа 
# восстановление из общего бэкапа
psql -U <username> -d <confluence_database_name> -f <all-postgresql-dump.sql>
# восстановление из отдельного бэкапа базы только для confgluence
psql -d <confluence_database_name> -f <confluence-dump.sql>
```

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


# Install Seafile pro 7.1.6

```bash
# Get Seafile Pro 7.1.6 package and put in `/home` directory

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
openssl dhparam -out /etc/nginx/dhparam.pem 4096

service nginx restart

# -------------------------------------------
# MariaDB
# -------------------------------------------

apt-get install -y mariadb-server
service mysql start

# установить пароль root можно так
mysqladmin -u root password <root-password> 
# либо в настройках безопасного подключения к БД
sudo mysql_secure_installation 	# на удаленное подключение по root отвечаем No

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
mv /home/seafile-pro-server_7.1.6_x86-64_Ubuntu.tar.gz /home/seafile/seafile-pro-server_7.1.6_x86-64_Ubuntu.tar.gz
tar xzf seafile-pro-server_7.1.6_x86-64_Ubuntu.tar.gz
mv /home/seafile/seafile-pro-server_7.1.6_x86-64_Ubuntu.tar.gz /home/seafile/installed/seafile-pro-server_7.1.6_x86-64_Ubuntu.tar.gz

# C отключенным шелом и одноименной группой
useradd --system --comment "seafile" seafile --home-dir  /home/seafile -m -U -s /bin/false
# установим права на директорию пользователя
chown seafile:seafile -R /home/seafile
# задаем пароль пользователя
passwd seafile

# Предоставим Nginx, доступ в домашнюю директорию пользователя seafile
usermod -a -G seafile www-data 	# Добавить пользователя www-data в группу seafile

# gpasswd -d www-data seafile - Удалить пользователя www-data из группы seafile
# id www-data - Проверить в какие группы входит пользователь www-data

cd /home/seafile/seafile-pro-server-7.1.6

# -------------------------------------------
# Create ccnet, seafile, seahub conf using setup script
# -------------------------------------------
# more on https://seafile.gitbook.io/seafile-server-manual/deploying-seafile-under-linux/deploying-seafile-with-mysql

./setup-seafile-mysql.sh auto -u seafile -w <mysql-seafile-password> -r <mysql-root-password> -p 17100

python3 /home/seafile/seafile-pro-server-7.1.6/pro/pro.py setup --mysql --mysql_host=127.0.0.1 --mysql_port=3306 --mysql_user=seafile --mysql_password=Cj6yaRO#TWv6 --mysql_db=seahub_db

# -------------------------------------------
# Fix permissions
# -------------------------------------------

chown seafile:seafile -R /home/seafile

# -------------------------------------------
# Start seafile server
# -------------------------------------------

sudo -u seafile /home/seafile/seafile-pro-server-7.1.6/seafile.sh start
sudo -u seafile /home/seafile/seafile-pro-server-7.1.6/seahub.sh start 17101

# wait and stop

sudo -u seafile /home/seafile/seafile-pro-server-7.1.6/seafile.sh stop
sudo -u seafile /home/seafile/seafile-pro-server-7.1.6/seahub.sh stop

# -------------------------------------------
# Fix other permissions
# -------------------------------------------

chown seafile:seafile -R /tmp/seafile-office-output/

# -------------------------------------------
# Change seafdav.conf
# -------------------------------------------

cd ../conf
```

# Seafile configs [/home/seafile/conf]

## seafdav.conf

```ini
[WEBDAV]
enabled = true
port = 17200	# default 8080
fastcgi = false
share_name = /seafdav
```

## ccnet.conf

```ini
[General]
SERVICE_URL = https://cloud.ln

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
enabled = false 

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
SITE_BASE                           = 'https://cloud.ln'
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
CLOUD_MODE                          = True
ENABLE_WIKI                         = True

LOGIN_REMEMBER_DAYS                 = 30
LOGIN_ATTEMPT_LIMIT                 = 3

FILE_PREVIEW_MAX_SIZE               = 30 * 1024 * 1024
SESSION_COOKIE_AGE                  = 60 * 60 * 24 * 7 * 2
SESSION_SAVE_EVERY_REQUEST          = False
SESSION_EXPIRE_AT_BROWSER_CLOSE     = False

FILE_SERVER_ROOT                    = 'https://cloud.ln/seafhttp'
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
sudo -u seafile /home/seafile/seafile-pro-server-7.1.6/seafile.sh stop
sudo -u seafile /home/seafile/seafile-pro-server-7.1.6/seahub.sh stop

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

You can see the logs of the service using `journalctl -u seafile`

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

Темная тема оформления: https://github.com/u-alexey/seafile_ce-edition_custom_theme

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
	listen [::]:80;
	server_name  cloud.ln;
    return 301 https://$host$request_uri;
}

server {
	listen [::]:443 ssl http2;
	listen 443 ssl http2;
	ssl on;
	server_name cloud.ln;
	
	ssl_certificate /home/certs/cloud.ln/cloud.ln.crt;
	ssl_certificate_key /home/certs/cloud.ln/device.key;

	proxy_set_header X-Forwarded-For $remote_addr;
	client_max_body_size 0;
    
	location / {
		proxy_pass         http://17.10.1.1:17101;
		proxy_set_header   Host $host:$proxy_port;
		proxy_set_header   X-Real-IP $remote_addr;
		proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header   X-Forwarded-Host $server_name;
		proxy_set_header   X-Forwarded-Proto https;
         
		proxy_read_timeout  1200s;
	}
    
	location /seafhttp {
		rewrite ^/seafhttp(.*)$ $1 break;
		proxy_pass http://17.10.1.1:17100;
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
		proxy_request_buffering off;
	}
}
```

Добавим также следующее в nginx.conf:

```nginx
user www-data;
worker_processes auto;
worker_rlimit_nofile 100000;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 4000;
	use epoll;
	multi_accept on;
}

http {
	# cache informations about FDs, frequently accessed files
	# can boost performance, but you need to test those values
	open_file_cache max=200000 inactive=20s;
	open_file_cache_valid 30s;
	open_file_cache_min_uses 2;
	open_file_cache_errors on;
	# copies data between one FD and other from within the kernel
	# faster than read() + write()
	sendfile on;
	sendfile_max_chunk 128k;
	# send headers in one piece, it is better than sending them one by one
	tcp_nopush on;
	# don't buffer data sent, good for small data bursts in real time
	tcp_nodelay on;
	types_hash_max_size 2048;
	# Just For Security Reason
	server_tokens off;

	# buffer size for reading client request header -- for testing environment
	client_header_buffer_size 2k;
	# if the request body size is more than the buffer size, then the entire (or partial)
	# request body is written into a temporary file
	client_body_buffer_size 256K;
	client_max_body_size 12m;
	# maximum number and size of buffers for large headers to read from client request
	large_client_header_buffers 4 256k;
	
	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	# ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
	# Dropping SSLv3, ref: POODLE
	ssl_protocols TLSv1.2;
	ssl_prefer_server_ciphers on;
	# Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
	ssl_dhparam /etc/nginx/dhparam.pem;	
	ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES256-SHA384;
	ssl_ecdh_curve secp384r1;
	ssl_session_cache shared:SSL:10m;
	ssl_session_timeout 5m;
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";	
	##
	# Logging Settings
	##
	# to boost I/O on HDD we can disable access logs
	access_log off;
	# only log critical errors
	error_log /var/log/nginx/error.log crit;

	# default logging config:
	# access_log /var/log/nginx/access.log;
	# error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##
	# reduce the data that needs to be sent over network -- for testing environment
	gzip on;
	gzip_min_length 10240;
	gzip_comp_level 1;
	gzip_proxied any;
	gzip_vary on;
	gzip_disable msie6;
	gzip_proxied expired no-cache no-store private auth;
	gzip_types
        # text/html is always compressed by HttpGzipModule
		text/css
		text/javascript
		text/xml
		text/plain
		text/x-component
		text/cache-manifest
		text/vcard
		text/vnd.rim.location.xloc
		text/vtt
		text/x-cross-domain-policy
		application/javascript
		application/x-javascript
		application/json
		application/xml
		application/rss+xml
		application/atom+xml
		application/ld+json
		application/manifest+json
		application/vnd.geo+json
		application/vnd.ms-fontobject
		application/x-font-ttf
		application/x-web-app-manifest+json
		application/xhtml+xml
		font/truetype
		font/opentype
		image/bmp
		image/x-icon
		image/svg+xml;
	# allow the server to close connection on non responding client, this will free up memory
	reset_timedout_connection on;
	# request timed out -- default 60
	# read timeout for the request body from client -- for testing environment
	client_body_timeout 5;
	# how long to wait for the client to send a request header -- for testing environment
	client_header_timeout 3;
	# if client stop responding, free up memory -- default 60
	send_timeout 3;
	# server will close connection after this time -- default 75
	keepalive_timeout 30;
	# number of requests client can make over keep-alive -- for testing environment
	keepalive_requests 100000;

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
	listen [::]:80;
        server_name jupyter.ln;
	return 301 https://$host$request_uri;
}

server {
	listen [::]:443 ssl http2;
	listen 443 ssl http2;
	ssl on;
	server_name jupyter.ln;
	client_max_body_size 0;

	ssl_certificate /home/certs/jupyter.ln/jupyter.ln.crt;
	ssl_certificate_key /home/certs/jupyter.ln/device.key;	

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

## Установка консольного клиента Seafile (seaf-cli)

```
# добавим ключ для установки клиента seafile из репозитория
wget -O - http://linux-clients.seafile.com/seafile.key | sudo apt-key add -
# добавляем репозиторий
sudo bash -c "echo 'deb [arch=amd64] http://linux-clients.seafile.com/seafile-deb/buster/ stable main' > /etc/apt/sources.list.d/seafile.list"
# обновляемся
sudo apt update
# устанавливаем консольный клиент для seafile
sudo apt-get install -y seafile-cli

# создаем директорию для настроек
mkdir ~/seafile-client 
# инициализируем клиента с указанной директории настроек
seaf-cli init -d ~/seafile-client
# запускаем клиент
seaf-cli start
# проверяем статус клиента
seaf-cli status
# если используем самоподписанные сертификаты, то пропустим их верификацию
seaf-cli config -k disable_verify_certificate -v true
```

# Backups

## Добавляем домашнюю директорию jupyter в частное облако Seafile

По умолчанию домашней директорией для Jupyter была назначена /home/jupyter. Теперь нам надо установить seaf-cli (консольный клиент для seafile) и синхронизировать с новой библиотекой домашнюю директорию Jupyter, чтобы добавляемые файлы в Jupyter, сразу появлялись в нашем облаке. Тогда, взяв ссылку с облака, мы можем добавлять Jupyter Notebooks в Confluence посредством соответствующего плагина. 
Плюсом ко всему, мы можем синхронизировать облако с компьютером в созданной сети и т.п.
```
# создаем библиотеку в облаке seafile и смотрим её id в адресной строке браузера (символы через тире)
# синхронизируем домашнюю директорию jupyter
seaf-cli sync -l "the id of the library" -s  https://cloud.ln -d /home/jupyter -u artif467@gmail.com -p "password"
# синхронизация происходит автоматически, если запущен клиент seaf-cli
# для запуска клиента в определенное время или его остановки, воспользуемся cron
crontab -e
# добавим следующие задачи
30 22 * * * seaf-cli start
00 23 * * * seaf-cli stop
# стоит учесть, что клиент не запустится автоматически после перезагрузки компьютера (по умолчанию)
# посмотреть текущий статус можно так
seaf-cli status
# часто бывают проблемы с самоподписанными сертификатами..
```


## Форматируем внешний диск и создаем разделы

Для локального сервера используем дополнительный жесткий диск, на котором будем хранить резервные копии.
Изначально потребуется отформатировать внешний жесткий диск:
```
# определим внешний диск
lsblk
# более подробную информацию можем получить так
fdisk -l
# например, это диск sdb
# удалим все разделы на данном диске (осторожно и внимательно!)
fdisk /dev/sdb
# вводим d и нажимаем Enter, выбираем номер раздела, который хотим удалить
# при выходе, надо будет выполнить выбранные изменения, для чего нажимаем w
# далее создадим новый раздел
fdisk /dev/sdb
# вводим n для создания нового раздела
# вводим p для указания primary раздела
# вводим 1 для указания первого раздела
# вводим w для записи изменений
# теперь остается создать файловую систему и отформатировать диск
# узнать поддерживаемые в текущей системе файловые системы можно так
ls /sbin/mk*
# далее отмонтируем диск
umount /dev/sdb1
# создадим файловую систему
mkfs -t ext4 /dev/sdb1
```

Полезная опция, используемая для файловых систем ext2 и ext3 – это опция -L с именем, которое будет присвоено разделу в качестве метки. Эту метку можно использовать вместо имени устройства при монтировании файловой системы; при внесении определенных изменений, которые также необходимо отражать в управляющих файлах, метка позволяет изолировать имя устройства, которому она фактически соответствует. Для просмотра существующей метки файловой системы или создания новой используется команда e2label. Длина метки ограничена 16 символами.  Идентификаторы UUID обладают большей уникальностью по сравнению с метками и особенно полезны при использовании устройств с горячей заменой, например, USB-накопителей.
```
blkid /dev/sdb1
```

Теперь монтируем диск:
```
sudo mkdir /mnt/backups
sudo mount /dev/sdb1 /mnt/backups
# монтирование с использованием метки диска
sudo mount --label="ext1" /mnt/
```

Узнать доступное на диске место можно так:
```
df -h /dev/sdb1
```

## Резервное копирование баз данных PostgreSQL

```
# создаем директорию для бэкапов
mkdir /home/postgres_backups
chown postgres:postgres /home/postgres_backups
# создаем задание в cron
crontab -e -u postgres
# добавляем следующее
00 23 * * * pg_dumpall > /home/postgres_backups/pg_dump_all.sql > /home/postgres_backups/pg_dump.log
```

## Резервное копирование баз данных MariaDB (MySQL)

Для создания бэкапа потребуется создать директорию, а затем добавить в cron соответствующую команду:
```
mkdir /home/mysql_backups

# создадим файл с учетными данными - permissions 600
nano ~/.my.cnf
# добавим учетные данные следующим образом
[mysqldump]
user=mysqluser
password=secret
# пробуем создать дамп БД
mysqldump --all-databases --single-transaction --quick --lock-tables=false > /home/mysql_backups/dump-$(date +%F).sql -u root
# примонтируем внешний диск
mount /dev/sdb1 /mnt/backups
# создадим на внешнем диске требуемую директорию
mkdir /mnt/backups/mysql_backups
# добавим задачу синхронизации
rsync -aAXvh --delete /home/mysql_backups/* /mnt/backups/mysql_backups
# отмонтируем диск
umount /dev/sdb1

# теперь можно создать задачу в cron
crontab -e 
# добавляем следующее
50 22 * * * mysqldump --all-databases --single-transaction --quick --lock-tables=false > /home/mysql_backups/dump-$(date +%F).sql -u root
```


## Резервное копирование файлов

```
# примонтируем диск и создадим нужные директории на нем
mount /dev/sdb1 /mnt/backups
mkdir /mnt/backups/atlassian
mkdir /mnt/backups/certs
mkdir /mnt/backups/jupyter
mkdir /mnt/backups/seafile
mkdir /mnt/backups/mysql_backups
mkdir /mnt/backups/postgres_backups
mkdir /mnt/backups/nginx_configs

# создадим скрипт, который будем запускать через cron
nano /usr/local/bin/cron_local_backup.sh

# -----------------------------------------------------------------------
# добавим следующее
# -----------------------------------------------------------------------
#!/bin/bash

mount /dev/sdb1 /mnt/backups

rsync -aAXvh --delete /home/atlassian/* /mnt/backups/atlassian
rsync -aAXvh /home/certs/* /mnt/backups/certs
rsync -aAXvh --delete /home/jupyter/* /mnt/backups/jupyter
rsync -aAXvh --delete /home/seafile/* /mnt/backups/seafile
rsync -aAXvh --delete /home/mysql_backups/* /mnt/backups/mysql_backups
rsync -aAXvh --delete /home/postgres_backups/* /mnt/backups/postgres_backups
rsync -aAXvh /etc/nginx/sites-available/* /mnt/backups/nginx_configs
rsync -aAXvh /etc/nginx/nginx.conf /mnt/backups/nginx_configs

umount /dev/sdb1

# -----------------------------------------------------------------------

# дадим права на исполнение скрипта
chmod +x /usr/local/bin/cron_local_backup.sh
# добавим скрипт в cron для выполнения в 22-00 вечера, каждый день
crontab -e
50 22 * * * /usr/local/bin/cron_local_backup.sh
```

Теперь мы настроили резервное копирование Linux на подключенный внешний диск, причем удаление файлов в системе будет приводить к их удалению в резервной копии. Стоит заметить, что в данном случае, восстановление по ошибке удаленного файла, нужно произвести до начала обновления резервной копии, то есть до вечера текущего дня.

В Windows 10 достаточно настроить точки восстановления в Settings -> Recovery и Task Scheduler -> Microsoft -> Windows -> SystemRestore -> Edit -> Triggers, а затем, дополнительно создать образ всей настроенной системы, и делать инкрементальные копии посредством Acronis True Image. При необходимости, можно также создать диск восстановления через Acronis TI, либо можно использовать Windows 10 File History.


# Полезные команды
```
# вывод истории команд
history
# найти ранее введенную команду
history | grep -i <command_part>

# выполнение команды из истории
!<command number>
# выполнить предыдущую команду от root
sudo !!

# список задач в cron для конкретного пользователя
crontab -u <username> -l
# редактирование задач в cron для пользователя
crontab -u <username> -e
# тип команд для cron
<time_period * * * * *> <command>
[Minute] [hour] [Day_of_the_Month] [Month_of_the_Year] [Day_of_the_Week] [command]
Minute 0 – 59
Hour 0 – 23
Day of month 1 – 31
Month of year 1 – 12
Day of week 0 – 7
# текущая временная зона
timedatectl
# создание скрипта для запуска по cron
nano /usr/local/bin/cron_<script_name>.sh
# делаем скрипт исполняемым
chmod +x /usr/local/bin/cron_<script_name>.sh
# добавляем в cron
mm hh * * * /usr/local/bin/cron_<script_name>.sh

# примонтировать и отмонтировать диск
mount <drive_partition> <directory>
umount <drive_partition>
# пример
mount /dev/sdb1 /mnt
umount /dev/sdb1

# просмотреть диски
fdisk -l

# посмотреть свободное место в удобочитабельном виде
df -h <drive or directory>
# например
df -h /dev/sdb1

# резервное копирование файлов из одной директории в другую (внешний диск представляет собой примонтированное к опр. директории пространство)
rsync -aAXvh --delete <source_dir/file> <target_dir>
# опция --delete позволит удалять файлы, которые были удалены из source_dir 
# прогресс операции в процентах (сравнение)
# переходим в source_dir
cd <source_dir>
# выполняем следующее
rsync -a --info=progress2 <target_dir> .
# либо просто сравним объем исходной и целевой директории

# узнаем, свободен ли какой-то конкретный порт
sudo lsof -i -P -n | grep LISTEN
# отобразим список процессов, которые запущены прямо сейчас 
ps -ef
# поиск процесса можно осуществить с помощью grep
ps -ef | grep -i <name_part>
# отобразим список процессов пользователей
ps -f -u user1,user2
#  отобразим процессы потребляющие большое количество ресурсов ЦПУ
ps aux —sort=-pcpu,+pmem
# отобразим время прошедшее с запуска процесса
ps -e -o pid,comm,etime
```



## DynDNS

https://account.dyn.com/

