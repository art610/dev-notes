# Проверка Django и администрирование

В самом начале необходимо будет проверить наше развертывание Django на production сервере:

```bash
# проверка production сервера на уязвимости
su - djangouser
workon djangoenv
cdvirtualenv
cd djproject
python manage.py check --deploy
```

Вывод последней команды покажет необходимые изменения для повышения безопасности. Данный список изменений нужно соответствующим образом проработать. 

Далее приведен список команд для работы с сервером и службами:

```bash
# управление службами/сервисами
# перезагрузить системного демона Emperor (фоновый процесс)
systemctl daemon-reload
# перезагрузка uWSGI (также команды start/stop)
systemctl restart djproject.uwsgi.service 
# перезагрузка NGINX
sudo /etc/init.d/nginx restart

# управление и проверка конфигурации NGINX
nginx -t
sudo service nginx <start|stop|restart>
# перезапускаем веб-сервера с прокси
systemctl restart djproject.uwsgi.service
 sudo /etc/init.d/nginx restart

# работа с Django
# заходим за djangouser
su - djangouser
# запускаем виртуальное окружение
workon djangoenv

# директория проекта Django
/home/djangouser/.virtualenvs/djangoenv/djproject/

# файлы конфигурации uWSGI
/home/djangouser/.virtualenvs/djangoenv/djproject/djproject_uwsgi.ini

# файлы конфигурации NGINX
/etc/nginx/nginx.conf
/home/djangouser/.virtualenvs/djangoenv/djproject/djproject_nginx.conf
/etc/nginx/sites-available/

# пути к сертификату SSL
/etc/letsencrypt/live/lnovus.online/fullchain.pem
/etc/letsencrypt/live/lnovus.online/privkey.pem
```

