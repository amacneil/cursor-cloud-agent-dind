# any ubuntu or debian based image should work
FROM ubuntu:24.04

# Configure docker apt repo
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
    && rm -rf /var/lib/apt/lists/* \
    && install -m 0755 -d /etc/apt/keyrings \
    && . /etc/os-release \
    && curl -fsSL https://download.docker.com/linux/${ID}/gpg -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/${ID} ${VERSION_CODENAME} stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install development tools
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        docker-ce \
        docker-compose-plugin \
        git \
        git-lfs \
        less \
        python3 \
        python3-pip \
        ripgrep \
        sudo \
        tmux \
        unzip \
        wget \
        zip \
    && rm -rf /var/lib/apt/lists/*

# Give ubuntu user docker and sudo access
RUN usermod -aG docker ubuntu \
    && echo "ubuntu ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Switch user and configure environment
USER ubuntu
WORKDIR /home/ubuntu
CMD ["/bin/bash"]
