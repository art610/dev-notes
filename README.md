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

https://vk.com/doc363366274_550670627?hash=9b799060e0d6f814d3&dl=7899683d9158a3857a

Install python 3.7
https://stackoverflow.com/questions/61430166/python-3-7-on-ubuntu-20-04

http://wiki.ros.org/melodic/Installation/Ubuntu
