# Оптимизация веб-сервера NGINX

```bash
# редактируем файл конфигурации NGINX
sudo nano /etc/nginx/nginx.conf
# устанавливаем следующие значения
worker_connections 4000;

##
# `gzip` Settings
##

gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_min_length 256;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;

# для смены worker_rlimit_nofile можно выполнить системную команду
ulimit -n 200000
# или изменить файл /etc/security/limits.conf
```

