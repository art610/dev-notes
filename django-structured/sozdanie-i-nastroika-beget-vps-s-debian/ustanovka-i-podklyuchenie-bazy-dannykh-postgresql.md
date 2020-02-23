# Установка и подключение базы данных PostgreSQL

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

