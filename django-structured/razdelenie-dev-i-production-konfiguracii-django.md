# Разделение Dev и Production конфигураций Django

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

