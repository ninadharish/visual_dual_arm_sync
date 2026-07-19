FROM osrf/ros:jazzy-desktop

ARG USERNAME=dev
ARG USER_UID=1000
ARG USER_GID=1000

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    sudo \
    bash-completion \
    build-essential \
    cmake \
    git \
    gdb \
    curl \
    wget \
    nano \
    vim \
    libeigen3-dev \
    libopencv-dev \
    python3-colcon-common-extensions \
    python3-pip \
    python3-rosdep \
    python3-vcstool \
    ros-jazzy-rmw-fastrtps-cpp \
    ros-jazzy-cv-bridge \
    ros-jazzy-image-transport \
    ros-jazzy-vision-msgs \
    ros-jazzy-tf2-ros \
    ros-jazzy-tf2-geometry-msgs \
    ros-jazzy-rviz2 \
    ros-jazzy-xacro \
    ros-jazzy-robot-state-publisher \
    ros-jazzy-joint-state-publisher-gui \
    ros-jazzy-ur-description \
    ros-jazzy-ur-msgs \
    && rm -rf /var/lib/apt/lists/*

# Create a development user matching the host UID/GID.
RUN set -eux; \
    if getent group "${USER_GID}" >/dev/null; then \
        GROUP_NAME="$(getent group "${USER_GID}" | cut -d: -f1)"; \
    else \
        groupadd --gid "${USER_GID}" "${USERNAME}"; \
        GROUP_NAME="${USERNAME}"; \
    fi; \
    if getent passwd "${USER_UID}" >/dev/null; then \
        EXISTING_USER="$(getent passwd "${USER_UID}" | cut -d: -f1)"; \
        usermod --login "${USERNAME}" "${EXISTING_USER}"; \
        usermod --home "/home/${USERNAME}" --move-home "${USERNAME}" || true; \
        usermod --shell /bin/bash "${USERNAME}"; \
        usermod --gid "${GROUP_NAME}" "${USERNAME}"; \
    else \
        useradd \
            --uid "${USER_UID}" \
            --gid "${GROUP_NAME}" \
            --create-home \
            --shell /bin/bash \
            "${USERNAME}"; \
    fi; \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" \
        > "/etc/sudoers.d/${USERNAME}"; \
    chmod 0440 "/etc/sudoers.d/${USERNAME}"

RUN echo 'source /opt/ros/jazzy/setup.bash' \
        >> "/home/${USERNAME}/.bashrc" \
    && echo 'if [ -f /workspace/project/ros_ws/install/setup.bash ]; then source /workspace/project/ros_ws/install/setup.bash; fi' \
        >> "/home/${USERNAME}/.bashrc" \
    && chown -R "${USER_UID}:${USER_GID}" "/home/${USERNAME}"

USER ${USERNAME}

WORKDIR /workspace/project/ros_ws

CMD ["bash"]
