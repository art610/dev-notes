# Проектирование и документирование программных систем

Создаем ключ SSH для подключения к репозиторию на GitHub и присваиваем его текущему пользователю
```bash
# создаем новый ключ
yes ~/.ssh/id_rsa | ssh-keygen -q -t rsa -b 4096 -C "874803539@qq.com" -N '' > /dev/null
# добавляем ключи к пользователю
ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
# выводим публичный ключ
cat ~/.ssh/id_rsa.pub
```

Теперь необходимо перейти **в аккаунт на Github (можно создать новый аккаунт или использовать существующий)** и добавить выведенный в терминал публичный SSH-ключ в настройки аккаунта (Settings -> SSH and GPG keys), указав также необходимый заголовок.
Проверить возможность доступа можно командой:

```bash
# проверяем подключение к GitHub
sudo ssh -vT git@github.com
```

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDecotG4xHPk98+h1YgqfK0wSnSpYRHisIyCV12pzWjj20sLhWrWLFrds8Y9ERI8Ito12qhRJxQ1t3lo3YHvpDLZvbCilkJYHsF58xFWkO2hi9jun+anrdvQnktQ3C9oY36HpnMVEq98Qey87UsS6JeWpUf27QAcmXnJJyJyYV/DRZci5yz7bMb3Q5ULiwUtDLnYoCi/NYDLJ731EDQ1wp3yVnU/zVghOkonkRRloydOIm1pxhrCOf3K6ewOAu6RYGjm4F6ykeg2t+WJZFkIWVV2M4y96ydzAOUyb0OKJ2nS0/GFYjM0MqhMRObzYYeIi7RN6+EIYB5lafeTzotapiIpSZ09mvtMibJzbCMIB7va8y5bnAkyQPAAezT/3ZfHjg3L7MPB8kRPb87M73cnz/xghr+JCxco7VpJ2VXn6J47W4UuLqYAQgBelLP+l3t+3JXtkJMGnXB3Y7GVzCZF/NV+lM6WvlC01EjAb03dQaP+RzHg9BLsuVhsN8fWe4S+OI3L5uXT7glqtIH8ZZKyYpj45ayNxsDR2NY//ZA1BCUPOo8RKk3zSEEmxMn+/sKuYUT2KAQo6fP0O2Hl+fCDN5JITnvjVwhD3Ny/eCG8HPUoYINlSvKXKW/FGON0s2g7PntBrKwGfDn/D7EGcU/POykGVWRQXQlKEbjDIMF97DDww== 874803539@qq.com
```
