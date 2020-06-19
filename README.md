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


Install ROS on Ubuntu18.4
```
    5  sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
    6  sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
    7  sudo apt-get update
    8  sudo apt update
    9  sudo apt install ros-noetic-desktop-full
   10  ros
   11  sudo apt install ros-melodic-ros-base
   12  sudo rosdep init
   13  python2 -V
   14  python -V
   15  python3 -V
   16  pip -V
   17  pip3 -V
   18  pip2 -V
   19  sudo apt-get install python pip
   20  sudo apt-get install python-pip
   21  sudo apt-get install python3-pip
   22  source /opt/ros/melodic/setup.bash
   23  echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
   24  source ~/.bashrc
   25  roscore
   26  history
```

http://wiki.ros.org/melodic/Installation/Ubuntu

https://www.theconstructsim.com/how-to-install-ros-on-ubuntu/
