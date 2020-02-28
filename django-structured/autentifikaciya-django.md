---
description: >-
  Создание шаблонов и обработчиков для пользовательской формы входа с
  использованием Bootstrap 4
---

# Аутентификация Django

Для начала необходимо перейти в директорию проекта Django и создать новое приложение:

```bash
# за djangouser заходим в директорию проекта под вирт. окр.
su - djangouser
cd /home/djangouser/.virtualenvs/djangoenv/djproject
workon djangoenv
# создаем приложение core (будет создан каркас)
python manage.py startapp core
# добавим новое приложение в INSTALLED_APPS
sudo nano djproject/settings/base.py
# В итоге приводим INSTALLED_APPS к следующему виду
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core.apps.CoreConfig',    # новое приложение
]
```

Настроем перенаправление и создадим роутер внутри приложения:

```python
# редактируем глобальный роутер 
sudo nano djproject/urls.py


# djproject/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('core.urls')), # add core/urls.py
]



# создаем роутер внутри приложения
sudo nano core/urls.py


# добавляем направление на index
# core/urls.py
from django.urls import path
from . import views    # импортируем файл views.py

# перенаправляем главную страницу на функцию index во views.py
urlpatterns = [
    path('', index, name='index') 
]
```

Создадим простое представление views.py для приложения:

```python
sudo nano core/views.py

# core/views.py
from django.http import HttpResponse

# функция индекс будет возвращать простой текст
def index(request):
    return HttpResponse("Server working, but we havn't interfaces")
```

Далее создадим папку с шаблонами для проекта:

```python
cd /home/djangouser/.virtualenvs/djangoenv/djproject
sudo mkdir templates

# в settings/base.py добавим директорию в TEMPLATES
sudo nano djproject/settings/base.py

# отредактируем TEMPLATES до следующего вида
# добавив 'DIRS': [os.path.join(BASE_DIR, 'templates')],
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'django.template.context_processors.media',
            ],
        },
    },
]

# создадим базовый шаблон в папке templates
sudo nano templates/base.html
```



![Joxy Image](../.gitbook/assets/dde8330730-1582885072506.png)

