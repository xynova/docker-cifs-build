FROM alpine
MAINTAINER hec <https://github.com/xynova>
RUN apk update && apk add curl \
&& mkdir -p /tmp/imgops && cd /tmp/imgops \
&& curl -O -L https://xynova+testrobot:FMH7W81TEYOFQ8KGU8JBLSIBKL7CNAW8192664T651BHJLGRD8IKS5SPN6VV7E8Z@quay.io/c1/squash/xynova/test/latest \
&& cd /tmp/imgops && tar -ztf latest | grep .tar | xargs tar -zxvf latest $1 | xargs tar -xv sbin/mount.cifs -f $1 \
&& cp sbin/mount.cifs /sbin \
&& rm -Rf /tmp/imgops && apk del curl
