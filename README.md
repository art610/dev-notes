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

Uninstall all ROS packages:
sudo apt-get remove ros-*

Install ROS from sources files:
```
   29  sudo apt-get install python-rosdep python-rosinstall-generator python-wstool build-essential
   30  sudo rosdep init
   31  rosdep update
   32  mkdir ~/ros_catkin_ws
   33  cd ~/ros_catkin_ws
   34  rosinstall_generator desktop --rosdistro melodic --deps --tar > melodic-desktop.rosinstall
   35  vcs import src < melodic-desktop.rosinstall
   36  sudo apt-get install vcs
   37  sudo apt-get install python3-vcstool
   38  vcs import src < melodic-desktop.rosinstall
   39  ls
   40  mkdir src
   41  vcs import src < melodic-desktop.rosinstall
   42  rosdep install --from-paths src --ignore-src --rosdistro melodic -y
   43  sudo rosdep install --from-paths src --ignore-src --rosdistro melodic -y
   44  sudo apt-get install -y python-opencv
   45  ps aux | grep -i apt
   46  sudo kill -9 11751
   47  sudo kill -9 11755
   48  sudo kill -9 11996
   49  ps aux | grep -i apt
   50  sudo kill -9 12026
   51  ps aux | grep -i apt
   52  rosdep install --from-paths src --ignore-src --rosdistro melodic -y
   53  sudo killall apt apt-get
   54  ls /var/lib/apt/
   55  ls /var/lib/apt/lists/
   56  rosdep install --from-paths src --ignore-src --rosdistro melodic -y
   57  sudo apt-get update
   58  rosdep install --from-paths src --ignore-src --rosdistro melodic -y
   59  sudo rm /var/lib/dpkg/lock-frontend
   60  sudo dpkg --configure -a
   61  sudo rm /var/lib/dpkg/lock
   62  sudo rm /var/cache/apt/archives/lock
   63  sudo rm /var/lib/apt/lists/lock
   64  sudo dpkg --configure -a
   65  clear
   66  rosdep install --from-paths src --ignore-src --rosdistro melodic -y
   67  ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
   68  source ~/ros_catkin_ws/install_isolated/setup.bash
   69  roscore

```
