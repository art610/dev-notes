# Разделение файла конфигурации Django

Чтобы создать настройки для  разработки dev.py и для production сервера production.py мы разнесем файл settings.py на три файла в отдельно созданной папке. 

Создаем папку settings в директории с файлом settings.py, затем переносим файл settings.py в папку settings и переименовываем его в base.py \(наши базовые настройки Django\). 

Создаем в папке settings файл dev.py \(настройки на время разработки\) и файл production.py \(настройки для сервера\).

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

В директории settings создаем файл dev.py `sudo nano settings/dev.py` и заносим в него конфигурацию в соответствии с файлом **`dev.py`**

Далее также создаем файл production.py `sudo nano settings/production.py` и заносим в него конфигурацию в соответствии с файлом **`production.py`**

Редактируем файл base.py `sudo nano settings/base.py`. Далее необходимо закоментировать строку с секретным ключом SECRET\_KEY,  а все остальные строки в файле base.py привести в соответствие с конфигурацией, представленной в отдельном файле **base.py**

Теперь изменим ссылки на файлы конфигурации в manage.py и в wsgi.py:

```bash
sudo nano \
/home/djangouser/.virtualenvs/djangoenv/djproject/manage.py

sudo nano \
/home/djangouser/.virtualenvs/djangoenv/djproject/djproject/wsgi.py

sudo nano \
/home/djangouser/.virtualenvs/djangoenv/djproject/djproject/asgi.py
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

