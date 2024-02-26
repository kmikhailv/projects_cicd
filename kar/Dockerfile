FROM redmine.rtc.ru:5000/astra AS build
COPY conf/ssh.tar.gz /tmp

FROM redmine.rtc.ru:5000/astra

RUN groupmod -n kar-pa klavesin
RUN usermod -G kar-pa,fuse,astra-admin -d /home/kar-pa -p kar-pa -m -l kar-pa klavesin
RUN chown kar-pa.kar-pa -R /home/kar-pa
RUN usermod  kar-pa -p $(openssl passwd -crypt "kar-pa")

COPY --from=build /tmp/ssh.tar.gz /tmp
RUN tar -xf /tmp/ssh.tar.gz -C /home/kar-pa && rm /tmp/ssh.tar.gz

RUN wget https://redmine.rtc.ru:8181/scm/hg/vityaz-bin/raw-file/cb56924034c9/depends/deb/cmake-3.9.6-Linux-x86_64.tar.gz --user=koval --password=1232425355 -O /tmp/cmake.tar.gz && tar --strip-components=1 -xf /tmp/cmake.tar.gz cmake-3.9.6-Linux-x86_64/ -C /usr/local/ && rm /tmp/cmake.tar.gz && apt-get remove cmake -y
RUN wget http://redmine.rtc.ru/files/astra1.5/install/pool/main/libc/libconfig/libconfig++-dev_1.4.8-5_amd64.deb && \
    wget http://redmine.rtc.ru/files/astra1.5/install/pool/main/libc/libconfig/libconfig++8-dev_1.4.8-5_amd64.deb && \
    wget http://redmine.rtc.ru/files/astra1.5/install/pool/main/libc/libconfig/libconfig-dev_1.4.8-5_amd64.deb && \
    wget http://redmine.rtc.ru/files/astra1.5/install/pool/main/libc/libconfig/libconfig-doc_1.4.8-5_all.deb && \
    wget http://redmine.rtc.ru/files/astra1.5/install/pool/main/libc/libconfig/libconfig8-dev_1.4.8-5_amd64.deb && \
    dpkg -i libconfig*deb && rm libconfig*deb

RUN mkdir -p /logs && chmod 777 /logs
RUN mkdir -p /var/run/screen && chmod 777 /var/run/screen
