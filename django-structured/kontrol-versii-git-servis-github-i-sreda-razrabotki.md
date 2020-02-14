# Контроль версий Git и веб-сервис GitHub

Все файлы конфигурации \(nginx/uwsgi\) необходимо занести в .gitignore, также как и настройки production.py и файл unix-сокета. 

Берем из основной папки файл **manage.py** и папки со статикой и медиа **static/** **media/**, из папки с настройками проекта файлы **settings.py**, **urls.py** и **wsgi.py**. При создании приложений также берем основные файлы из них \(models.py, urls.py, views.py и подобные\).

```bash
# переходим в основную папку проекта и инициализируем репозиторий
cd /home/djangouser/.virtualenvs/djangoenv/djproject/
git init

# создаем .gitignore
sudo nano .gitignore 
```

Добавляем в .gitignore следующее:

```bash
### Django ###
*.log
*.pyc
db.sqlite3
djproject_nginx.conf
djproject_uwsgi.ini
djproject/settings/production.py
*.pyo
*~
media/**
**/__pycache__/*
*.py[cod]

# C extensions
*.so

# PyInstaller
#  Usually these files are written by a python script from a template
#  before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Translations
*.mo 
*.pot

# PyBuilder
target/

# Unit test / coverage reports
htmlcov/
.tox/
.coverage
.cache
nosetests.xml
coverage.xml

# Sphinx documentation
docs/_build/

# Thumbnails
._*
favicon.ico

# IDE
.idea/
```

Устанавливаем глобальные параметры Git:

```bash
# настроим глобальные параметры git
git config --global user.name "ProductionServer01"
git config --global user.email "admin@alistorya.com"
git config --global core.pager "less -r"
```

### Добавление в новый репозиторий

Если мы хотим добавить проект в чистый репозиторий, который только что создали, то выполняем следующее:

```bash
git add .

# проверим, какие файлы и папки мы теперь отслеживаем
git status

# при необходимости удаляем папки и файлы из индекса
# без удаления самого файла --cached
git rm --cached <filename>
git rm -r --cached <foldername>

# делаем инициализирующий коммит
git commit -m "Initial commit"

# подключаем удаленный репозиторий
git remote add origin <ssh-link>

# отправляем файлы в удаленный репозиторий
git push -f origin master
```

### Получаем данные из удаленного репозитория

Если в удаленном репозитории содержатся файлы проекта Django, с которыми мы работали ранее, то нам необходимо их получить и выполнить слияние с файлами на сервере. Для этого выполняем следующее:

```bash
# переходим в основную папку проекта и инициализируем репозиторий
cd /home/djangouser/.virtualenvs/djangoenv/djproject/
# подключаем удаленный репозиторий
git remote add origin <ssh-link>
# получаем все изменения
git fetch --all
# проводим сброс
git reset --hard origin/master
# если находимся на другой ветке
git reset --hard origin/<branch_name>
# применяем последние изменения
git pull origin master
```

Команда `git fetch` получает последние файлы с удаленного репозитория без замены и слияния локальных файлов, а затем мы сбрасываем ветку до той, которую получили из удаленного репозитория, причем опция --hard заменит все файлы в рабочей ветке, которые совпадут с файлами из удаленного репозитория \(подобно --force\). 

Далее можно выполнить соответствующие команды в окружении Django:

```bash
# вход за djangouser
su - djangouser
# активируем виртуальное окружение и переходим в директорию проекта
workon djangoenv
cdvirtualenv
cd djproject

# выполняем необходимые операции
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic
python manage.py createsuperuser

# перезапускаем все службы
sudo systemctl daemon-reload && sudo /etc/init.d/nginx restart && systemctl restart djproject.uwsgi.service
```

### Настройка контроля версий

Для контроля версий используем связку Git-GitHub и три ветки, помимо master, а именно:

* **dev** - в данной ветке ведем разработку \(пока работаем одни и приложение имеет небольшие размеры, а в будущем можно её поделить на ветки featrure, bugfix/hotfix и т.п.\). Данную ветку сливаем в master;
* **master** - основная ветка при разработке - если есть несколько разработчиков, то изменения они сливают в master, где далее проводят тесты и прочие операции, чтобы отправить код на ветку stage;
* **stage** - ветка для pre-production тестирования и отладки, которая выкатывается на промежуточный сервер и после выполнения всех необходимых операций сливается в production;
* **production** - ветка для выкладки на основной сервер, которому имеют доступ пользователи. 

```bash
# просмотреть данные git
git status    # состояние проекта
git log    # журналы проекта

# работа с ветками репозитория
git branch -a    # показать все ветки репозитория
git branch stage    # создать новую ветку с названием new_feature
git branch    # текущая ветка

git checkout new_feature    # перейти на созданную ветку
git checkout master    # перейти на основную ветку проекта
git merge new_feature    # произвести слияние ветки new_feature с основной

# стандартные операции
# добавим все файлы проекта в локальный репозиторий
sudo git add . 
# сделаем коммит в локальный репозиторий
sudo git commit -m 'message'
# запушим изменения в удаленный репозиторий
sudo git push -u origin master 
# получим обновления из удаленного репозитория
sudo git pull origin 
```

Теперь мы можем работать в ветке dev и при окончании сливать её в master, где можно проводить дополнительные операции, либо сливать туда параллельно другие ветки из разработки и тестирования для разрешения конфликтов, а затем её сливать в stage при помощи GitHub PullRequest, а после отладки на stage сервере и разрешения конфликтов можно выкатывать изменения на ветку production также при помощи GitHub PullRequest.

