FROM debian
MAINTAINER hec <https://github.com/xynova>
RUN apt-get update && apt-get install -y build-essential curl \
&& curl https://ftp.samba.org/pub/linux-cifs/cifs-utils/cifs-utils-6.4.tar.bz2 | bunzip2 -c - | tar -xvf - \
&& cd cifs-utils-6.4 && ./configure && make && cp mount.cifs /sbin/ \
&& apt-get remove -y build-essential curl && apt-get -y autoremove && apt-get clean
