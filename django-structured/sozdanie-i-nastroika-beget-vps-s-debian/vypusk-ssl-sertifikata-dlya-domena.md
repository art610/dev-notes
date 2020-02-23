# Выпуск SSL сертификата для домена

Запускаем certbot с данными эл. почты \(указать вместо &lt;admin@mail.com&gt;\) и домена \(указать вместо &lt;domain.com&gt;, также потребуется добавить к DNS домена последовательно две записи TXT с указанным кодом acme, подождать 15 минут и отправить на одобрение запрос\):

```bash
certbot certonly --preferred-challenges=dns --agree-tos \
-m <admin@mail.com> -d *.<domain.com> -d <domain.com> --manual \
--server https://acme-v02.api.letsencrypt.org/directory
```

Выводим данные о сертификате и ключе \(необходимо получить путь к файлам\):

```bash
echo -e "\033[32m $(certbot certificates) \033[0m"
```

Полученные данные сертификата будут представлены следующим образом:

```bash
Certificate Name: <domain.com>
    Domains: *.<domain.com> <domain.com>
    Expiry Date: <somedate> (VALID: n days)
    Certificate Path: /etc/letsencrypt/live/<domain.com>/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/<domain.com>/privkey.pem
```

Теперь необходимо правильно настроить конфигурацию nginx и перезапустить службы: обновим зависимости и отредактируем файл конфигурации NGINX:

```bash
# обновляем зависимости
sudo apt-get -y update && apt-get -y dist-upgrade
sudo nano \
/home/djangouser/.virtualenvs/djangoenv/djproject/djproject_nginx.conf
```

Окончательный вид файла конфигурации NGINX представлен в файле **djproject\_nginx.conf**. При использовании данной конфигурации необходимо заменить &lt;domain.com&gt; на подключенный домен.

Перезагрузка и проверка конфигурации:

```bash
nginx -t
sudo /etc/init.d/nginx restart
systemctl restart djproject.uwsgi.service
# при необходимости добавим домен в ALLOWED_HOSTS в файле settings.py
```

Закрываем 8000 порт при помощи ufw:

```bash
ufw deny 8000/tcp
```

Теперь сайт можно открыть только если указать домен. Соответственно остается доступ на портах 22 для ssh подключения и 443 для https \(с 80 порта происходит перенаправление на https\).

