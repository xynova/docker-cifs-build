# Samba cifs binary Docker build

The Dockerfiles in this repository provide an automated build for the mount.cifs binary to enable Azure File Store mounts on package-manager-less distributions like [CoreOS](https://coreos.com).

Pull and Run the **quay.io/xynova/cifs-mount** image (docker pull quay.io/xynova/cifs-mount) and then copy mount.cifs (/sbin/mount.cifs) binary onto the host. You can then use that to provide cifs enabled volumes for other containers. 

````
mkdir /mnt/azureshare
/tmp/test/mount.cifs -o vers=2.1,username=xxx,password=xxx,dir_mode=0777,file_mode=0777 //xxx.file.core.windows.net/sharexxx /mnt/azureshare
````

Get more info on how to mount the FS at the  [MS Azure's website](https://azure.microsoft.com/en-us/documentation/articles/storage-how-to-use-files-linux/)


## Build process
The main build process is the following:

````
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
````
Unfortunately, I could not get this process to nicely build with the Alpine distro so I separated it into 2 steps:

* **STEP1:** The "make" using a Debian (~140MB) based image that gets automatically built at **Quay.io:** [![Docker Repository on Quay.io](https://quay.io/repository/xynova/cifs-build/status "Docker Repository on Quay.io")](https://quay.io/repository/xynova/cifs-build).

* **STEP2:** The "packaging" of the resulting mount.cifs binary into an Alpine image, also automatically built at **Quay.io**: [![Docker Repository on Quay.io](https://quay.io/repository/xynova/cifs-mount/status "Docker Repository on Quay.io")](https://quay.io/repository/xynova/cifs-mount)
