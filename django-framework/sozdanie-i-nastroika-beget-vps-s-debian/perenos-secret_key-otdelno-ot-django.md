# Перенос SECRET\_KEY

Для того, чтобы не выдавать секретный ключ при удаленном хранении кода, его необходимо поместить в переменную системного фонового процесса, и делать вызов непосредственно от туда. Значение SECRET\_KEY ранее мы оставили в закоментированной строке в файле settings/base.py, поэтому открываем данный файл командой `sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/djproject/settings/base.py`   
и копируем значение **SECRET\_KEY\_VALUE**. 

{% hint style="danger" %}
Если SECRET\_KEY содержит символ %, то его стоит экранировать повтором % \(получится так %%\). Одинарные кавычки также экранируют другие служебные символы bash.
{% endhint %}

```bash
# в settings/production.py пропишем вызов переменной окружения
SECRET_KEY = os.environ['SECRET_KEY_VALUE']

# заходим под пользователем <DJANGO_USER> и устанавливаем для него 
# системную переменную окружения
su - djangouser
export SECRET_KEY_VALUE='*******************************************'
# добавим его также в .bashrc
sudo nano ~/.bashrc
# в конец файла добавляем следующую строку
export SECRET_KEY_VALUE='*******************************************'
# обращаем внимание на одинарные кавычки 
# они экранируют служебные символы Bash
# выходим за root
exit
# передаем переменную системному сервису
sudo nano /etc/systemd/system/djproject.uwsgi.service
# добавляем туда --env таким образом
ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/apps-enabled \
--uid <DJANGO_USER> --env SECRET_KEY_VALUE='<value>'
# пароль к базе данных мы храним в отдельном файле настроек production.py
# причем удаленный доступ к БД закрыт (порт закрыт)

# перезапускаем демона
systemctl daemon-reload
# перезапускаем веб-сервера
systemctl restart djproject.uwsgi.service
sudo /etc/init.d/nginx restart
```

Затем также открываем base.py командой   
`sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/djproject/settings/base.py`   
и удаляем ранее закоментированную строку c SECRET\_KEY. Теперь наш секретный ключ и пароль к базе данных будут всегда оставаться на сервере, где им самое место. Главное файлы конфигурации и настроек сервера исключать из контроля версий и т.п.

