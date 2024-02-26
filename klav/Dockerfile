FROM ubuntu:xenial AS build

RUN apt-get update && apt-get install -y wget 
RUN wget https://redmine.rtc.ru:8181/scm/hg/klav-bin/raw-file/cb56924034c9/depends/deb/pylon-5.1.0.12682-x86_64.tar.gz -O /tmp/pylon.tar.gz --user=koval --password=123124235245
RUN tar -xf /tmp/pylon.tar.gz -C /tmp
RUN mkdir -p /tmp/pylon && tar -xf /tmp/pylon-5.1.0.12682-x86_64/pylonSDK-5.1.0.12682-x86_64.tar.gz -C /tmp/pylon
COPY conf/ssh-client.tar.gz /tmp
COPY conf/ssh-server.tar.gz /tmp
COPY conf/vnc-server.tar.gz /tmp


FROM ubuntu:xenial

RUN apt-get update && apt-get install -y curl build-essential qt5-default libopencv-dev libboost-all-dev libcurl4-gnutls-dev iputils-ping mc ssh mercurial sshfs sudo screen libconfig++-dev ntpdate vnc4server libpcap-dev psmisc

RUN apt-get install wget && wget https://redmine.rtc.ru:8181/scm/hg/klav-bin/raw-file/67f73f556a2f/depends/deb/libmodbus_2019.01.10-1_amd64.deb --user=koval --password=123124235245 && dpkg -i libmodbus_2019.01.10-1_amd64.deb && rm libmodbus_2019.01.10-1_amd64.deb


# for newest cmake 
#RUN apt-get install -y  software-properties-common && add-apt-repository -y ppa:george-edison55/cmake-3.x && apt update apt install cmake
RUN wget https://redmine.rtc.ru:8181/scm/hg/klav-bin/raw-file/cb56924034c9/depends/deb/cmake-3.9.6-Linux-x86_64.tar.gz --user=koval --password=123124235245 -O /tmp/cmake.tar.gz && tar --strip-components=1 -xf /tmp/cmake.tar.gz cmake-3.9.6-Linux-x86_64/ -C /usr/local/ && rm /tmp/cmake.tar.gz

RUN apt-get install -y qt5-default libqwt-headers libqt5svg5-dev

COPY --from=build /tmp/pylon/ /opt

RUN useradd -d /home/klav -c klav -m -p $(openssl passwd -crypt klav) klav
RUN usermod -G sudo klav
RUN echo "%sudo  ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

COPY --from=build /tmp/ssh-client.tar.gz /tmp
COPY --from=build /tmp/ssh-server.tar.gz /tmp
COPY --from=build /tmp/vnc-server.tar.gz /tmp

RUN tar -xf /tmp/ssh-client.tar.gz -C /home/klav && rm /tmp/ssh-client.tar.gz
RUN tar -xf /tmp/ssh-server.tar.gz -C /etc && rm /tmp/ssh-server.tar.gz
RUN tar -xf /tmp/vnc-server.tar.gz -C /home/klav && rm /tmp/vnc-server.tar.gz

RUN echo '[extensions]' >> /etc/mercurial/hgrc
RUN echo 'largefiles=' >> /etc/mercurial/hgrc

RUN mkdir /logs && chmod 777 /logs



