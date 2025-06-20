FROM ghcr.io/rtarun1/noetic:v1.0

LABEL org.opencontainers.image.description="Husky"
LABEL org.opencontainers.image.title="HuskyDevCon"
LABEL org.opencontainers.image.vendor="Tarun R"
LABEL org.opencontainers.image.source="https://github.com/rtarun1/husky_ws"
LABEL maintainer="tarun.ramak@gmail.com"
LABEL org.opencontainers.image.licenses="MIT"

# Use bash as default shell
SHELL ["/bin/bash", "-c"]

# ---- Fix ROS repository and expired GPG key ----
RUN apt-get update && apt-get install -y curl gnupg && \
    echo "deb http://packages.ros.org/ros/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" > /etc/apt/sources.list.d/ros-latest.list && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -

# ---- Basic dependencies ----
RUN apt-get update && apt-get install -y --no-install-recommends \
    libssl-dev \
    libusb-1.0-0-dev \
    libudev-dev \
    pkg-config \
    libgtk-3-dev \
    wget \
    cmake \
    build-essential \
    libglfw3-dev \
    libgl1-mesa-dev \
    libglu1-mesa-dev \
    at \
    libaria-dev \
    v4l-utils \
    xterm \
    libeigen3-dev \
    libpcl-dev \
    python3-rosdep \
    git \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# ---- Install required ROS packages (including octomap_msgs) ----
RUN apt-get update && apt-get install -y \
    ros-noetic-realsense2-camera \
    ros-noetic-pcl-ros \
    ros-noetic-navigation \
    ros-noetic-robot-state-publisher \
    ros-noetic-xacro \
    ros-noetic-joy \
    ros-noetic-octomap-msgs \
    ros-noetic-octomap \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# ---- Optional: install extra packages from rosPkgs.list if provided ----
COPY rosPkgs.list /tmp/rosPkgs.list
RUN apt-get update && xargs -a /tmp/rosPkgs.list apt-get install -y --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# ---- librealsense SDK ----
WORKDIR /home/container_user
RUN git clone --branch v2.56.2 https://github.com/IntelRealSense/librealsense.git
WORKDIR /home/container_user/librealsense
RUN mkdir -p /etc/udev/rules.d && \
    ./scripts/setup_udev_rules.sh && \
    ./scripts/patch-realsense-ubuntu-lts-hwe.sh
RUN mkdir build && cd build && \
    cmake ../ && \
    sudo make uninstall && make clean && make -j$(nproc) && sudo make install

# ---- Default development workspace ----
WORKDIR /workspaces/husky_ws
