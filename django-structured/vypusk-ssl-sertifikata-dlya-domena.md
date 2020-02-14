# Выпуск SSL сертификата для домена

Запускаем Certbot с данными эл. почты и домена \(потребуется добавить к DNS домена запись TXT с указанным кодом acme, подождать 15 минут и отправить на одобрение запрос\):

```bash
certbot certonly --preferred-challenges=dns --agree-tos -m admin@alistorya.com -d *.alistorya.com -d alistorya.com --manual --server https://acme-v02.api.letsencrypt.org/directory
```

Выводим данные о сертификате и ключе \(необходимо получить путь к файлам\):

```bash
echo -e "\033[32m $(certbot certificates) \033[0m"
```

Полученные данные:

```bash
Certificate Name: alistorya.com
    Domains: *.alistorya.com alistorya.com
    Expiry Date: 2020-05-12 11:41:34+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/alistorya.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/alistorya.com/privkey.pem
```

Теперь необходимо правильно настроить конфигурацию nginx и перезапустить службы: Обновим зависимости и отредактируем файл конфигурации NGINX:

```bash
# обновляем зависимости
sudo apt-get -y update && apt-get -y dist-upgrade
sudo nano /home/djangouser/.virtualenvs/djangoenv/djproject/djproject_nginx.conf
```

Окончательный вид файла конфигурации NGINX следующий:

```bash
upstream djproject {
	server unix:///home/djangouser/.virtualenvs/djangoenv/djproject/djproject.sock;
}
server {
  listen 80;
  listen [::]:80;
	server_name lnovus.online www.lnovus.online;
	return 301 https://$server_name$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl on;
        ssl_certificate /etc/letsencrypt/live/lnovus.online/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/lnovus.online/privkey.pem;

        server_name www.lnovus.online;
        return 301 $scheme://lnovus.online;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        charset     utf-8;
        
        ssl on;
        ssl_certificate /etc/letsencrypt/live/lnovus.online/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/lnovus.online/privkey.pem;
	
	      server_name lnovus.online;
	      client_max_body_size 75M;
	
				# путь к favicon.ico
        location = /favicon.ico {
                alias /home/djangouser/.virtualenvs/djangoenv/djproject/favicon.ico;
        }

	      
		    location /static {
								autoindex on;
	      	      alias /home/djangouser/.virtualenvs/djangoenv/djproject/static/;
	      }
	
				location /media  {
								autoindex on;
	      	      alias /home/djangouser/.virtualenvs/djangoenv/djproject/media/;
	      }
	
	      location / {
	      	      uwsgi_pass  djproject;
	      	      include     /etc/nginx/uwsgi_params;
	      }
}
```

Перезагрузка и проверка конфигурации:

```bash
nginx -t
sudo /etc/init.d/nginx restart
systemctl restart djproject.uwsgi.service
# при необходимости добавим домен в ALLOWED_HOSTS
```

Закрываем 8000 порт при помощи ufw:

```bash
ufw deny 8000/tcp
```

Теперь сайт можно открыть только если указать домен. Соответственно остается доступ на портах 22 для ssh подключения и 443 для https \(с 80 порта происходит перенаправление на https\).

