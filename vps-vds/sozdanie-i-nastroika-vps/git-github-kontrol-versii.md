# Git/GitHub - контроль версий



```bash
# Подключаем сервер к GitHub
# генерируем SSH ключ
yes ~/.ssh/id_rsa | ssh-keygen -q -t rsa -b 4096 -C "admin@alistorya.com" -N '' > /dev/null
# добавляем ключи к пользователю
ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
# выводим публичный ключ, который нужно скопировать и добавить в аккаунт на GitHub (Settings -> SSH and GPG keys)
cat ~/.ssh/id_rsa.pub
# затем проверяем подключение к GitHub
sudo ssh -vT git@github.com
```

