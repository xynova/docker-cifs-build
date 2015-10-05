FROM alpine
MAINTAINER hec <https://github.com/xynova>
RUN apk update && apk add curl \
&& mkdir -p /tmp/imgops && cd /tmp/imgops \
&& curl -O -L https://xynova+reader:SFZ78VTCO2TNUL6NHTBAWZBU68U1LPCT7EQ26B1ID6MIP3FZOFYRLXSF331WEQT4@quay.io/c1/squash/xynova/cifs-build/latest \
&& cd /tmp/imgops && tar -ztf latest | grep .tar | xargs tar -zxvf latest $1 | xargs tar -xv sbin/mount.cifs -f $1 \
&& cp sbin/mount.cifs /sbin \
&& rm -Rf /tmp/imgops && apk del curl
