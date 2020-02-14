# Первичная настройка сервера Debian

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

