# Первичная настройка Debian 10

## Основная конфигурация

После конфигурирования доступа по SSH и создания системного пользователя переходим к настройке операционной системы и установке требуемых пакетов. Дальнейшие действия производим непосредственно на удаленном сервере Debian при подключении по SSH через утилиту PuTTY \(или терминал Linux\). 

В самом начале добавим возможность получения пакетов из non-free репозиториев, для этого необходимо отредактировать файл sources.list. Однако на Beget VPS данный файл создается автоматически и может быть затем перезаписан, ввиду этого  необходимо изменить параметр`apt_preserve_sources_list: true` в файле `/etc/cloud/cloud.cfg`, для этого используем следующие команды:

```text
# открываем файл редактором nano для добавления параметра
sudo nano /etc/cloud/cloud.cfg
# добавим в конец файла параметр apt_preserve_sources_list: true
# сохраним изменения
# редактируем файл sources.list (добавляем contrib non-free для каждого репозитория)
sudo nano /etc/apt/sources.list
# сохраняем изменения: Ctrl+X затем Y и Enter
```

Следует привести файл `/etc/apt/sources.list`к следующему виду:

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
cd
# обновляем зависимости и пакеты
apt-get -y update && apt-get -y dist-upgrade
# устанавливаем необходимые пакеты одной командной
sudo apt-get -y install libtiff5-dev libjpeg62-turbo-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev gcc g++ libc-dev curl man libffi-dev libssl-dev libbz2-dev libncursesw5-dev libgdbm-dev liblzma-dev libsqlite3-dev tk-dev uuid-dev libreadline-dev build-essential libncurses5-dev libnss3-dev wget ufw git 
# nginx не включен в предыдущую команду, т.к. рекомендуется собрать его из исходников
# обновляем зависимости и пакеты
apt-get -y update && apt-get -y dist-upgrade

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

# установим python 3.8.1 - последняя версия на данный момент
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

# обновляем зависимости
sudo apt-get -y update && apt-get -y dist-upgrade

# выводим основную информацию
cat /etc/os-release    # информация о релизе ОС
arch    # архитектура ОС
nodejs -v
node -v
npm -v
git --version && python -V && pip -V
which python
which python3.8
```

### Установка NGINX из исходников

Пакет nginx доступен в прекомпилированном виде для любого дистрибутива. Идеально собирать NGINX из исходников самостоятельно и сделать его более компактным и надежным.

```bash
# переходим в /opt
cd /opt
# получаем исходники
wget https://github.com/nginx/nginx/archive/release-1.17.8.tar.gz
# распаковываем содержимое полученного архива
tar xf release-1.17.8.tar.gz
# переходим в папку с исходниками
cd nginx-release-1.17.8
```

Изменим строку приветствия Web-сервера, чтобы выдавать поменьше информации посторонним. Для этого скачиваем исходники Nginx, открываем в них файл `src/http/ngx_http_header_filter_module.c` и находим следующие строки:

```bash
static char ngx_http_server_string[] = "Server: nginx" CRLF;
static char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
```

Заменяем их так, чтобы не показывать название веб-сервера:

```bash
static char ngx_http_server_string[] = "Server: ][ Web Server" CRLF;
static char ngx_http_server_full_string[] = "Server: ][ Web Server" CRLF;
```

Удаляем все неиспользуемые модули \(выполняем сборку, исключая [ненужные модули](https://nginx.org/ru/docs/configure.html)\). Отключая ненужные модули мы снижаем вероятность того, что со временем будет найдена уязвимость:

```bash
# посмотрим модули
auto/configure --help
# конфигурируем
auto/configure \
--without-http_autoindex_module --without-http_ssi_module \
--with-mail_ssl_module --with-http_ssl_module --with-http_v2_module --with-http_realip_module
# запускаем сборку
make -s && make -s install
```

Так получаем nginx с заранее отключенными \(и в большинстве случаев бесполезными\) модулями SSI \(Server Side Includes\) и Autoindex. Чтобы узнать, какие модули можно безболезненно выбросить из Web-сервера, запустим скрипт configure с флагом --help, также справка доступна на [официальном сайте](https://nginx.org/ru/docs/configure.html).

Если требуется установить стандартную версию NGINX, то используем пакеты:

```bash
sudo apt-get install nginx
```

### Добавление SSL бота

```bash
# переходим в директорию по умолчанию
cd ~
# Установим дополнительные пакеты certbot для SSL/TLS
sudo apt install -y certbot python3-certbot-nginx
```

