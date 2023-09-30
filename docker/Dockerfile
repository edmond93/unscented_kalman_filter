FROM nvidia/cuda:12.2.0-devel-ubuntu22.04

# set default env variables
ENV DEBIAN_FRONTEND=noninteractive
ENV ROS_DISTRO=humble

RUN apt-get update && apt-get install -y locales locales-all
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# to set the timezone automatically
# here is source: https://serverfault.com/questions/683605/docker-container-time-timezone-will-not-reflect-changes
ENV TZ=Asia/Singapore
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# install essential build tools 
RUN apt-get update && \
    apt-get install -y gettext \
    software-properties-common \
    vim git cmake unzip \
    && rm -rf /var/lib/apt/lists/*

# install ros humble
RUN add-apt-repository universe

# add apt repo for ros humble
RUN apt update && apt install curl -y 
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

RUN apt update && \ 
    apt install -y ros-humble-desktop ros-dev-tools

# Install apt packages
COPY ./apt-packages /tmp/
RUN apt-get update && \
    apt-get install -y \
        $(cat /tmp/apt-packages | cut -d# -f1 | envsubst) \
    && rm -rf /var/lib/apt/lists/* /tmp/apt-packages

#  Install python requirements 
COPY ./requirements.txt .
RUN pip3 install -r requirements.txt

# Add user
ENV USERNAME robot
ARG USER_ID=1000
ARG GROUP_ID=15214
# ENV PULSE_SERVER /run/pulse/native

RUN groupadd --gid $GROUP_ID $USERNAME && \
    useradd --gid $GROUP_ID -m $USERNAME && \
    echo "$USERNAME:$USERNAME" | chpasswd && \
    usermod --shell /bin/bash $USERNAME && \
    usermod -aG sudo $USERNAME && \
    usermod  --uid $USER_ID $USERNAME && \
    echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

# For the user to have access to video (realsense and camera streams)
RUN usermod -a -G video $USERNAME

# Setup default user & entry points
COPY ./entrypoint.sh /tmp/entrypoint.sh

USER $USERNAME
ENTRYPOINT ["/tmp/entrypoint.sh"]
