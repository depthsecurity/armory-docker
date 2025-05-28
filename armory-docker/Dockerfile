FROM debian:stable-slim

# Install Armory build dependencies
RUN  \
    apt update && \
    apt install -y --no-install-recommends \
    default-libmysqlclient-dev \
    gcc \
    git \
    libmariadb-dev-compat \
    libxml2-dev \
    libxslt-dev \
    python3 \
    python3-dev \
    python3-pip \
    python3-virtualenv \
    libjpeg-dev \
    zlib1g-dev \
    libjpeg-dev \
    libfreetype6-dev \
    ca-certificates \
    curl \
    fonts-liberation

RUN pip install --break-system-packages -U setuptools
RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
RUN chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
RUN echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update
RUN apt-get install -y --no-install-recommends \
    docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# Clone Armory
WORKDIR /tmp
RUN git clone https://github.com/depthsecurity/armory
WORKDIR /tmp/armory
RUN git checkout docker
# WORKDIR /tmp/armory
# RUN git checkout fix-main-image
# Build Armory wheel
WORKDIR /tmp/armory-wheel
RUN pip wheel -r /tmp/armory/requirements.txt -w requirements && \
    pip wheel jinja2 xlsxwriter -w extras && \ 
    pip wheel django_q -w extras --no-deps && \
    pip wheel /tmp/armory/. -w core --no-deps

# # Exported Image
# FROM debian:testing-slim
# LABEL maintainer="dlawson@depthsecurity.com"
# # Copy Armory from build image
# COPY --from=build /tmp/armory-wheel /tmp/armory-wheel

# Install Armory
WORKDIR /tmp/armory-wheel
RUN /bin/bash -c "virtualenv /opt/armory/local/ && \
    source /opt/armory/local/bin/activate && \
    pip install requirements/* && \
    pip install extras/* && \
    pip install core/* && \
    pip install pycryptodome && \
    rm -rf /tmp/armory-wheel"

# Install misc pip packages
# RUN pip install pycryptodome

# Prepare launch_api.sh
COPY launch_api.sh /usr/bin/launch_api.sh
RUN chmod +x /usr/bin/launch_api.sh

COPY armory_launch.sh /usr/bin/armory
COPY armory_launch.sh /usr/bin/armory2
COPY armory_manage.sh /usr/bin/armory2-manage
COPY armory_manage.sh /usr/bin/armory-manage
COPY armory_shell.sh /usr/bin/armory-shell
COPY armory_shell.sh /usr/bin/armory2-shell
RUN chmod +x /usr/bin/armory /usr/bin/armory2 /usr/bin/armory2-manage /usr/bin/armory-manage /usr/bin/armory2-shell /usr/bin/armory-shell
# Cleanup
RUN pip cache purge && \
    rm -rf /root/.cache/* && \
    rm -rf /root/.local/* && \
    rm -rf /tmp/*

WORKDIR /root/current
