# Samba cifs binary Docker build

The Dockerfiles in this repository are part of an automated build for the Samba mount.cifs binary to enable [Azure File Store mounts](https://azure.microsoft.com/en-us/blog/azure-file-storage-now-generally-available/) on package-manager-less distributions like [CoreOS](https://coreos.com).

Get more info on how to mount Azure File Shares at the  [MS Azure's website](https://azure.microsoft.com/en-us/documentation/articles/storage-how-to-use-files-linux/).

##Tinkering with it

### Manually try it out
Run the **quay.io/xynova/cifs-mount** container to get a copy of the mount.cifs binary onto the host. This can then be used to provide cifs enabled volumes for other containers. 

```shell
# Get mount.cifs binary into /opt/bin 
mkdir -p /opt/bin
docker run --rm -v /opt/bin:/tmp/dir \
quay.io/xynova/cifs-mount sh -c "cp /sbin/mount.cifs /tmp/dir/"

# Mount an Azure File Share
mkdir /mnt/azureshare
/opt/bin/mount.cifs \
-o vers=2.1,username=xxx,password=xxx,dir_mode=0777,file_mode=0777 \
 //xxx.file.core.windows.net/sharexxx /mnt/azureshare
```

### -or- setup through systemd unit files
To get the mount.cifs binary into your distro:

* Copy the [basic-mount-cifs.service](https://github.com/xynova/docker-cifs-build/blob/master/coreos-setup/basic-mount-cifs.service) unit into `/etc/systemd/system`.
* Enable the unit with `systemctl enable basic-mount-cifs.service` to make sure the unit runs before the `docker.service` starts.
* Start the unit if you want to immediateley force the binary download `systemctl start basic-mount-cifs.service`

### -or- setup through CoreOS cloud-config
The [worker-cloud-config-example](https://github.com/xynova/docker-cifs-build/blob/master/coreos-setup/worker-cloud-config-example) provides an example on how to include the previously mentioned `basic-mount-cifs.service` systemd unit as part of a cloud-config init file.

## How is it built?
The entire source code pull and build process transparently occurs at [quay.io](https://quay.io/). This happens to be a pretty cool container repository brought to us from the CoreOS guys.

After several failed attempts at building the cifs binary source using an Alpine container image ([why?](http://wiki.alpinelinux.org/wiki/Running_glibc_programs)), I decided then to break the process into two steps: One that nicely fetches and builds the source code using a Debian based container, and then a second one that packages the mount.cifs binary into an Alpine image, ending up in only 6MB in size instead of the extra ~100MB of the Debian image.

### Step1
The [build-on-debian](https://github.com/xynova/docker-cifs-build/blob/master/build-on-debian/Dockerfile) Dockerfile is automatically pulled and built into the **quay.io/xynova/cifs-build** repository.

[![Docker Repository on Quay.io](https://quay.io/repository/xynova/cifs-build/status "Docker Repository on Quay.io")](https://quay.io/repository/xynova/cifs-build)

```shell
# Install build packages:
apt-get update && apt-get install -y build-essential curl

# Download and extract the source code from samba.org:
curl https://ftp.samba.org/pub/linux-cifs/cifs-utils/cifs-utils-6.4.tar.bz2 | bunzip2 -c - | tar -xvf - 

# Build the code:
cd cifs-utils-6.4 && ./configure && make 

# Copy the binary into the /sbin folder:
cp mount.cifs /sbin/ \

# Cleanup
apt-get remove -y build-essential curl && apt-get -y autoremove && apt-get clean
```
* [Check it out here](https://quay.io/repository/xynova/cifs-build/image/3e2dc75bf285ad707fe1d533c62f7a8df054e7ce6d8374a69b8b1c162b991614)

###Step 2
The [package-into-alpine](https://github.com/xynova/docker-cifs-build/blob/master/package-into-alpine/Dockerfile) Dockerfile is then pulled and built into the **quay.io/xynova/cifs-mount** repository. This takes the [squashed version](http://docs.quay.io/guides/squashed-images.html) image built in Step 1 and extracts the mount.cifs binary contained within it.
 
 [![Docker Repository on Quay.io](https://quay.io/repository/xynova/cifs-mount/status "Docker Repository on Quay.io")](https://quay.io/repository/xynova/cifs-mount)

```shell
# Install packages
apk update && apk add curl

# Create a temporary working directory
mkdir -p /tmp/imgops 

# Pull the "squash" image into the container 
cd /tmp/imgops && curl -O -L https://xynova+reader:SFZ78VTCO2TNUL6NHTBAWZBU68U1LPCT7EQ26B1ID6MIP3FZOFYRLXSF331WEQT4@quay.io/c1/squash/xynova/cifs-build/latest 

# Extract the mount.cifs binary using "tar"
cd /tmp/imgops && tar -ztf latest | grep .tar | xargs tar -zxvf latest $1 | xargs tar -xv sbin/mount.cifs -f $1 

# Copy the mount.cifs into "/sbin"
cp sbin/mount.cifs /sbin 

# Cleanup
rm -Rf /tmp/imgops && apk del curl
```
* [Check it out here](https://quay.io/repository/xynova/cifs-mount/image/76a9825651a02f1524798694858068c51f1804e152cddd3ca92508492055ecc8)