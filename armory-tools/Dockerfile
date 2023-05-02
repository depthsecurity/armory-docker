FROM debian:testing-slim as build

# Install build packages
RUN echo "deb-src http://deb.debian.org/debian buster main" >> /etc/apt/sources.list
RUN apt update && \
    apt install -y --no-install-recommends \
    build-essential \
    ca-certificates \ 
    gcc \
    git \
    make \
    wget \
    zlib1g-dev 


# Install Golang
ENV GOROOT "/tmp/golang/go"

WORKDIR /tmp/golang
RUN wget https://dl.google.com/go/go1.20.2.linux-$(dpkg --print-architecture).tar.gz -Ogo.linux.tar.gz && \ 
    tar zxf go.linux.tar.gz && \
    rm go.linux.tar.gz && \
    ln -s /tmp/golang/go/bin/go /usr/local/bin/ && \
    ln -s /tmp/golang/go/bin/gofmt /usr/local/bin/

# Install Go-based tooling

# Install Amass
RUN go install -v github.com/owasp-amass/amass/v3/...@master

# Install FFuF
RUN go install github.com/ffuf/ffuf@latest 

# Install Gobuster
RUN go install github.com/OJ/gobuster/v3@latest

# Install Gowitness
RUN go install github.com/sensepost/gowitness@latest

# Install Hakrawler
RUN go install github.com/hakluke/hakrawler@latest

# Install subfinder
RUN go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Install tko-subs
RUN go install github.com/anshumanbh/tko-subs@latest

# Clone tooling with git
WORKDIR /tmp/build
RUN git clone https://github.com/robertdavidgraham/masscan
RUN git clone https://github.com/rbsec/sslscan

# Compile masscan
WORKDIR /tmp/build/masscan
RUN make -j && make install

# Compile sslscan
WORKDIR /tmp/build/sslscan
RUN apt-get build-dep -y openssl && make static

# Exported Image
FROM debian:testing-slim as final
LABEL maintainer="dlawson@depthsecurity.com"
# Install necessary packages
RUN apt update && \
    apt install -y --no-install-recommends \
    chromium \
    chromium-driver \
    curl \
    dnsrecon \
    fonts-liberation \
    git \
    hydra \
    libimage-exiftool-perl \
    libnet-ssleay-perl \
    libpcap-dev \
    nano \
    nmap \
    p7zip-full \
    python3 \
    python3-pip \
    python3-venv \
    python3-bs4 \
    redis-server \
    sudo \
    tmux \
    unzip \
    whois \
    wget \
    vim \
    xclip 

# Upgrade Python pip and symlink Python3 to Python
RUN python3 -m pip install --upgrade pip --break-system-packages && \
    ln -s /usr/bin/python3 /usr/bin/python

# Install Python 3 virtualenv
RUN pip install virtualenv --break-system-packages

WORKDIR /opt/src

# Install tooling from GitHub

## Install ADIDNSDump 
RUN git clone --depth 1 https://github.com/absolomb/adidnsdump && \
    cd adidnsdump && \
    pip install . --break-system-packages

## Install Asnlookup
RUN git clone --depth 1 https://github.com/fang0654/Asnlookup && \
    cd Asnlookup && \
    ln -s /opt/src/Asnlookup/asnlookup.py /usr/local/bin/asnlookup.py && \
    chmod +x asnlookup.py

## Install DomLink
RUN git clone --depth 1 https://github.com/fang0654/DomLink && \
    cd DomLink && \
    ln -s /opt/src/DomLink/domLink.py /usr/local/bin/domLink.py && \
    chmod +x domLink.py

## Install ExploitDB
RUN git clone --depth 1 https://gitlab.com/exploit-database/exploitdb && \
    cd exploitdb && \
    ln -s /opt/src/exploitdb/searchsploit /usr/local/bin/searchsploit && \
    sed -i 's/\/opt\/exploitdb/\/opt\/src\/exploitdb/g' .searchsploit_rc && \
    cp .searchsploit_rc /root/

## Install Jexboss
RUN git clone --depth 1 https://github.com/joaomatosf/jexboss && \
    cd jexboss && \
    pip install -r requires.txt --break-system-packages && \
    ln -s /opt/src/jexboss/jexboss.py /usr/local/bin/jexboss.py

## Install LinkedInt
RUN git clone --depth 1 https://github.com/fang0654/LinkedInt && \
    cd LinkedInt && \
    touch /tmp/linkedint2 && \
    ln -s /opt/src/LinkedInt/LinkedInt.py /usr/local/bin/LinkedInt.py && \
    chmod +x LinkedInt.py

## Install nikto
RUN git clone --depth 1 https://github.com/sullo/nikto && \
    ln -s /opt/src/nikto/program /opt/nikto && \
    ln -s /opt/nikto/nikto.pl /usr/local/bin/nikto

## Install pymeta
RUN pip install git+https://github.com/m8r0wn/pymeta.git --break-system-packages

## Install Sublist3r
RUN git clone --depth 1 https://github.com/aboul3la/Sublist3r && \
    cd Sublist3r && \
    pip install -r requirements.txt --break-system-packages && \
    ln -s /opt/src/Sublist3r/sublist3r.py /usr/local/bin/sublist3r

## Install theHarvester
RUN git clone --depth 1 https://github.com/laramies/theHarvester && \
    cd theHarvester && \
    pip install -r requirements.txt --break-system-packages && \
    python3 setup.py install

## Install xsscrapy
RUN git clone --depth 1 https://github.com/L1NT/xsscrapy && \
    cd xsscrapy && \
    pip install -r requirements.txt --break-system-packages && \
    chmod +x xsscrapy.py

## Install XSStrike
RUN git clone --depth 1 https://github.com/s0md3v/XSStrike && \
    cd XSStrike && \
    pip install -r requirements.txt --break-system-packages && \
    chmod +x xsstrike.py && \
    ln -s /opt/src/XSStrike/xsstrike.py /usr/local/bin/xsstrike.py

# Cleanup unnecessary files
RUN apt clean -y && \
    apt autoremove -y && \
    rm -rf /var/lib/apt/lists/* && \
    pip cache purge  && \
    rm -rf /root/.cache

# Include Golang tooling from previous stage
COPY --from=build /root/go/bin/amass /usr/local/bin/amass
COPY --from=build /root/go/bin/ffuf /usr/local/bin/ffuf
COPY --from=build /root/go/bin/gobuster /usr/local/bin/gobuster
COPY --from=build /root/go/bin/gowitness /usr/local/bin/gowitness
COPY --from=build /root/go/bin/hakrawler /usr/local/bin/hakrawler
COPY --from=build /root/go/bin/subfinder /usr/local/bin/subfinder
COPY --from=build /root/go/bin/tko-subs /usr/local/bin/tko-subs
COPY --from=build /root/go/pkg/mod/github.com/anshumanbh/tko-subs*/providers-data.csv /usr/local/bin/providers-data.csv

# Include compiled tooling from previous stage
COPY --from=build /tmp/build/masscan/bin/masscan /usr/local/bin/masscan
COPY --from=build /tmp/build/sslscan/sslscan /usr/local/bin/sslscan

WORKDIR /