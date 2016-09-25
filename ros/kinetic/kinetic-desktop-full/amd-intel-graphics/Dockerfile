FROM ubuntu:xenial

MAINTAINER Jack Liu jacknlliu@gmail.com

# setup environment
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

# setup sources.list
RUN echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list

# setup keys
RUN apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 0xB01FA116

# update the repository
RUN apt-get update

# install ros packages
ENV ROS_DISTRO kinetic
RUN apt-get install -y ros-kinetic-desktop-full

# rosdep init
RUN rosdep init

# enables you to easily download many source trees for ROS packages with one command
RUN apt-get install -y python-rosinstall

# install additional system packages and ros packages
# update gazebo
RUN echo "deb http://packages.osrfoundation.org/gazebo/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/gazebo-latest.list

RUN curl http://packages.osrfoundation.org/gazebo.key |apt-key add -

RUN apt-get update && apt-get upgrade -y gazebo

# download gazebo models
RUN mkdir -p /ros_data/gazebo
RUN cd /ros_data/gazebo/ && hg clone https://bitbucket.org/osrf/gazebo_models

# chmod gazebo models directory
RUN chmod -R 777 /ros_data/gazebo/gazebo_models

# install ros related components
# RUN apt-get install -y ros-kinetic-moveit-full
RUN apt-get install -y  ros-kinetic-gazebo-ros-control \
    ros-kinetic-ros-controllers


# install essential tools
RUN apt-get install -y  bash-completion wget vim git terminator

# install additional build tool
RUN apt-get install -y build-essential gdb

# install IDE
RUN apt-get install -y mesa-common-dev libglu1-mesa-dev libfontconfig1
RUN apt-get install -y qt5-default qtcreator qtdeclarative5-qtquick2-plugin

# install ros qtcreator plugin
# RUN apt-get install -y software-properties-common python-software-properties
# RUN add-apt-repository -y ppa:beineri/opt-qt57-xenial
# RUN add-apt-repository -y ppa:levi-armstrong/ppa
# RUN apt-get update && apt-get install -y  qt57creator-plugin-ros

# install ssh and sudo
RUN apt-get install -y supervisor openssh-server  sudo

RUN mkdir -p /var/run/sshd
RUN echo "	ForwardX11Trusted yes" >> /etc/ssh/ssh_config

# install xterm fonts
RUN apt-get install -y xfonts-75dpi xfonts-100dpi xfonts-scalable

# install intel graphics driver, especially i915 driver
RUN echo "deb http://ppa.launchpad.net/oibaf/graphics-drivers/ubuntu kinetic main" > /etc/apt/sources.list.d/intel-video-driver.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 957D2708A03A4626
RUN apt-get update && apt-get install -y  xserver-xorg-video-intel

# install amd graphics open source driver
RUN apt-get install -y   xserver-xorg-video-ati mesa-utils

# apt-get autoclean
RUN apt-get autoclean -y \
&& apt-get autoremove -y \
&& rm -rf /var/lib/apt/lists/*

# copy supervisord.conf file
COPY ./supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# copy entrypoint file
COPY ./ros_entrypoint.sh /

# set user ros and sudo
RUN adduser --gecos "ROS User" --home /home/ros --disabled-password ros
RUN usermod -a -G dialout ros
RUN echo "ros ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/99_aptget

# switch to user ros, but the HOME is still /, not /home/ros
USER ros

# setup ros env
RUN HOME=/home/ros rosdep update
RUN echo "export QT_X11_NO_MITSHM=1" >> /home/ros/.bashrc
RUN echo "export HOME=/home/ros" >> /home/ros/.bashrc
RUN echo "source "/opt/ros/$ROS_DISTRO/setup.bash"" >> /home/ros/.bashrc

# link to ros user home directory
RUN mkdir -p /home/ros/.gazebo/ && sudo ln -s /ros_data/gazebo/gazebo_models /home/ros/.gazebo/models

# configure Qt
RUN mkdir -p /home/ros/.config/QtProject
COPY ./QtCreator.ini  /home/ros/.config/QtProject/

CMD ["bash"]
ENTRYPOINT ["/ros_entrypoint.sh"]