# Use Ubuntu 22.04 as the base image
FROM ubuntu:22.04

# Arguments for setting user ID and username
ARG UID=1000
ARG USER=ubuntu

# Set environment variables for running systemd in Docker
ENV container docker
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary packages
RUN apt-get update -y && \
    apt-get install -y systemd sudo net-tools curl wget jq python3-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a user and group with the specified UID
RUN groupadd -g ${UID} ${USER} && \
    useradd -u ${UID} -m -g ${USER} -s /bin/bash ${USER} && \
    echo "${USER} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/${USER} && \
    chmod 0440 /etc/sudoers.d/${USER}

# Set the working directory
WORKDIR /home/${USER}

# Set up the directories required for systemd
RUN mkdir -p /run/systemd && \
    echo 'docker' > /run/systemd/container

# Set the volume for cgroup and systemd runtime
VOLUME [ "/sys/fs/cgroup" ]

# Switch to non-root user
USER ${USER}

# Set default command to start systemd
CMD ["/sbin/init"]
