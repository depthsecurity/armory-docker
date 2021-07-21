FROM depthsecurity/armory-tools:latest
LABEL maintainer="dlawson@depthsecurity.com"

WORKDIR /opt/src
RUN git clone https://github.com/depthsecurity/armory.git
WORKDIR /opt/src/armory


RUN pip3 install django 
RUN python3 setup.py install
RUN pip3 install jinja2 xlsxwriter django_q
WORKDIR /root/current
COPY launch_api.sh /usr/bin/launch_api.sh
RUN chmod +x /usr/bin/launch_api.sh
