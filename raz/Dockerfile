FROM redmine.rtc.ru:5000/astra AS build

COPY libs/deb/pcl/ /tmp/pcl
COPY libs/deb/vlc /tmp/vlc
COPY libs/deb/qt-everywhere-opensource-src_5.9.8-1_amd64.deb /tmp/
COPY libs/cmake-3.9.6-Linux-x86_64.tar.gz /tmp/
COPY cfgs/ssh/ /tmp/ssh


FROM redmine.rtc.ru:5000/astra

COPY --from=build /tmp/pcl /tmp/pcl 
RUN dpkg -i /tmp/pcl/*.deb && rm -rf /tmp/pcl

COPY --from=build /tmp/vlc /tmp/vlc 
RUN dpkg -i /tmp/vlc/*.deb && rm -rf /tmp/vlc && apt-get remove gcc g++ -y

RUN echo /usr/local/lib64/ > /etc/ld.so.conf.d/libc64.conf && ldconfig

COPY --from=build /tmp/qt-everywhere-opensource-src_5.9.8-1_amd64.deb /tmp/ 
RUN dpkg -i /tmp/qt-everywhere-opensource-src_5.9.8-1_amd64.deb && rm -rf /tmp/qt-everywhere-opensource-src_5.9.8-1_amd64.deb

COPY --from=build /tmp/cmake-3.9.6-Linux-x86_64.tar.gz /tmp/ 
RUN tar --strip-components=1 -xf /tmp/cmake-3.9.6-Linux-x86_64.tar.gz cmake-3.9.6-Linux-x86_64/ -C /usr/local/ && rm /tmp/cmake-3.9.6-Linux-x86_64.tar.gz && apt-get remove cmake -y
RUN groupmod -n raz klav
RUN usermod -G raz,fuse -d /home/raz -p raz -m -l raz klav
RUN chown raz.raz -R /home/raz
RUN sed 's@session\s*required\s*pam_parsec_cap.so@session optional pam_parsec_cap.so@g' -i /etc/pam.d/su
RUN sed 's@session\s*required\s*pam_parsec_cap.so@session optional pam_parsec_cap.so@g' -i /etc/pam.d/login
COPY --from=build /tmp/ssh/known_hosts /home/raz/.ssh/
RUN chown  -R raz.raz /home/raz/.ssh/
#sed 's@INTERFACE\s=\s"br0"@INTERFACE = "eth0"@g' -i arm_rtk_r.cfg