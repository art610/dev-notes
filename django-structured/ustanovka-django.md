# Установка Django



Среда на production сервере будет представлена следующим образом:

![Django on Production Server](../.gitbook/assets/1562449042114.png)

То есть запрос от клиента будет поступать на HTTP Reverse Proxy Web-Server NGINX, затем через Unix-сокет запрос передается на uWSGI и далее по протоколу WSGI запрос клиента поступает фреймворку Django \(то есть к роутеру и дальше обрабатывается как необходимо\).

Создадим пользователя для работы с Django и соответствующим окружением:

```bash
# создаем системного пользователя <DJANGO_USER> 
# указываем любое имя пользователя, например, djangouser
# в ходе диалога указываем пароль, учетные данные оставим пустыми
# затем подтверждаем создание пользователя, нажав клавишу Y и Enter
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
# выйдем из виртуалньой среды и из-под root 
# также установим django глобально
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

# создадим тестовый файл и открываем его в редакторе
sudo nano test.py		

# в файл test.py добавляем следующее
def application(env, start_response):
	start_response('200 OK', [('Content-Type','text/html')])
	return [b"Hello World"] 
	
# проверяем подключение uWSGI из-под root, 
# проверяем в браузере доступ к <SERVER_IP>:8000 и затем выходим Ctrl+C
uwsgi --http :8000 --wsgi-file \
/home/djangouser/.virtualenvs/djangoenv/djproject/test.py

# запускаем созданный тестовый файл как веб-приложение через uWSGI 
# перезаходим за djangouser
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
# скачиваем и переименовываем любую картинку
wget  https://i.redd.it/jsnlg9kwvtxx.jpg
mv jsnlg9kwvtxx.jpg media.jpg
cd ..
uwsgi --http :8000 --module djproject.wsgi
# переходим по <server-ip>:8000/media/media.jpg
# если видим картинку, то всё работает
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
# переходим по <server-ip>/media/media.jpg и 
# если видим картинку, то всё работает
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
# Добавим параметр STATIC_ROOT в конец файла settings.py
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

# запускаем виртуальное окружение и переносим статику
workon djangoenv
cdvirtualenv
cd djproject
python manage.py collectstatic
deactivate
```

Создаем файл конфигурации для NGINX, соответственно выполняем команду `sudo nano djproject_nginx.conf` и добавляем конфигурацию в соответствии с файлом **first\_djproject\_nginx.conf**. Не забываем изменить &lt;SERVER\_IP&gt;.

Создаем симлинк для активации конфига и перезапускаем nginx:

```bash
sudo ln -s /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_nginx.conf \
/etc/nginx/sites-enabled/
# перезапускаем NGINX
sudo /etc/init.d/nginx restart
```



Проверим работу веб-сервера:

```bash
uwsgi --socket djproject.sock --wsgi-file test.py --chmod-socket=666
uwsgi --socket djproject.sock --module djproject.wsgi --chmod-socket=666
```

Создаем конфигурационный файл для uwsgi `sudo nano djproject_uwsgi.ini` и добавляем конфигурацию в соответствии с файлом **djproject\_uwsgi.ini**

Пробуем запустить uwsgi через файл с конфигурацией:

```bash
exit 		# [root]
uwsgi --ini \
/home/djangouser/.virtualenvs/djangoenv/djproject/djproject_uwsgi.ini \
--uid djangouser
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
sudo ln -s \
/home/djangouser/.virtualenvs/djangoenv/djproject/djproject_uwsgi.ini \
/etc/uwsgi/apps-enabled/
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

В данный файл добавляем конфигурацию в соответствии с файлом **djproject.uwsgi.service**

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

