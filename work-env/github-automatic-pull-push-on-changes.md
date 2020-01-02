---
description: >-
  Automatic repository updates with Git-o-matic using Windows-Git-Github-Gitbook
  chain.
---

# Github Automatic Pull/Push on changes

Установить Git на локальный компьютер и настроить подключение по SSH к GitHub аккаунту. Ключ будет расположен в &lt;id\_rsa\_path&gt;.

Установить [Go Lang Env](https://golang.org/doc/install) для сборки из исходников Gitomatic \([git-o-matic](https://github.com/muesli/gitomatic/blob/master/README.md)\). Получить пакеты и собрать данную утилиту.

```text
git clone https://github.com/muesli/gitomatic.git
cd gitomatic
go build
```

После сборки полученную директорию сохранить и использовать при запуске путь до файла gitomatic &lt;gitomatic\_path&gt;, либо занести путь в переменные среды Windows.

Инициализировать новый, либо получить один из старых репозиториев. Перейти в папку данного репозитория и установить внешнюю ссылку на него:

```text
git clone git@github.com...
cd <local_repo_dir>

git remote set-url --add origin git@github.com...
git remote set-url --delete origin https://...
```

Создать bat-файл и конвертировать его в exe \(64-bit, без отображения консоли и запроса на права администратора\):

```text
@echo off

<gitomatic_path>\gitomatic -privkey <id_rsa_path>/git_id_rsa <local_repo_dir>    # without ending slash
```

Полученный exe-файл добавляем в папку на автозагрузку \(просто переносим его туда\):

```text
Win+R # "Выполнить"
shell:startup    # откроет папку программ на автозагрузку
```

Если использовать совместно с GitHub также GitBook для заметок, то создание заметок нужно производить непосредственно на GitBook, иначе новый файл не будет отображаться на GitBook, а будет доступен лишь в репозитории на GitHub и локально.

Из удаленного репозитория на GitBook стоит получить все изменения до начала работы \(git pull\) иначе возможен конфликт. В данной системе также не предполагается одновременная работа с локальными файлами и Gitbook, т.к. подобное может также привести к конфликту. Стоит использовать GitBook для внесения изначальных правок и затем лишь переносить всё в локальный репозиторий и, например, Marxico для публикации в личных блокнотах Evernote.

$$
a^2 = b + x^3
$$

