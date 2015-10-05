# Samba cifs binary Docker build

The Dockerfiles in this repository provide an automated build for the mount.cifs binary to enable Azure File Store mounts on package-manager-less distributions like [CoreOS](https://coreos.com).


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

* **STEP2:** The "packaging" of the resulting mount.cifs binary into an Alpine image, also automatically built at **Quay.io**:
