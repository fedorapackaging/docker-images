Distrobuilder: build LXC container images
=========================================

# References
* [This README](https://github.com/fedorapackaging/docker-images/blob/master/distrobuilder)
  is part of
  [the Docker images for Fedora packaging project](https://github.com/fedorapackaging/docker-images)
* [Distrobuilder on GitHub](https://github.com/lxc/distrobuilder)
* [Using distrobuilder to create container images for LXC and LXD](https://blog.simos.info/using-distrobuilder-to-create-container-images-for-lxc-and-lxd/)

# Installation

## Dependencies

### Debian / Ubuntu
```bash
$ aptitude -y install golang-go debootstrap rsync gpg squashfs-tools make gcc
```

## Go

### Linux
* Install a more recent version of Go:
```bash
$ mkdir -p /opt/go
$ wget https://dl.google.com/go/go1.12.2.linux-amd64.tar.gz
$ tar zxf go1.12.2.linux-amd64.tar.gz
$ mv go /usr/local/go-1.12.2
```

## All
* Configure Go:
```bash
$ cat >> ~/.bashrc << _EOF

# Go
export GOROOT="/usr/local/go-1.12.2"
export GOPATH="\${HOME}/go"
export PATH="${GOPATH}/bin:${GOROOT}/bin:${PATH}"

_EOF
$ . ~/.bashrc
```

* Download the source for `distrobuilder`:
```bash
$ go get -d -v github.com/lxc/distrobuilder
github.com/lxc/distrobuilder (download)
package github.com/lxc/distrobuilder: no buildable Go source files in /root/go/src/github.com/lxc/distrobuilder
```

* Compile `distrobuilder`:
```bash
$ pushd $HOME/go/src/github.com/lxc/distrobuilder
$ make
gofmt -s -w .
go get -t -v -d ./...
go install -v ./...
...
distrobuilder built successfully
$ popd
```

* Check that `distrobuilder` has been built correctly:
```bash
$ type distrobuilder
distrobuilder is /root/go/bin/distrobuilder
```

# Build a container image
* Build the image:
```bash
$ mkdir -p $HOME/container-images/fedora/
$ cd $HOME/container-images/fedora/
$ cp $HOME/go/src/github.com/lxc/distrobuilder/doc/examples/fedora fedora.yaml
$ distrobuilder build-lxc fedora.yaml
$ ls -lh
total 77M
-rw-r--r-- 1 root root 1.2K May  7 01:09 fedora.yaml
-rw-r--r-- 1 root root  512 May  7 01:26 meta.tar.xz
-rw-r--r-- 1 root root  49M May  7 01:27 rootfs.tar.xz
```

* Copy the image to the Proxmox template directory:
```bash
$ export TODAY_DATE=$(date "+%Y%m%d")
$ cp rootfs.tar.xz /vz/template/cache/fedora-30-default_${TODAY_DATE}_amd64.tar.xz
```

# Create a LXC container on a Proxmox
```bash
$ pct create 130 local:vztmpl/fedora-30-default_${TODAY_DATE}_amd64.tar.xz --arch amd64 --cores 8 --hostname fedf30.myowndomain.org --memory 65536 --swap 65536 --net0 name=eth0,bridge=vmbr1,gw=10.30.1.2,ip=10.30.1.130/24,type=veth --onboot 1 --ostype unmanaged
$ pct resize 130 rootfs 50G
```

* Bootstrap the network
```bash
$ pct start 130
$ pct enter 130
$ hostnamectl set-hostname fedf30.myowndomain.org
$ echo "fedf30.myowndomain.org" > /etc/hostname
$ cat > /etc/sysconfig/network << _EOF
NETWORKING=yes
HOSTNAME=fedf30.myowndomain.org
_EOF
$ ip addr add 10.30.1.130/24 dev eth0
$ ip link set eth0 up
$ ip route add default via 10.30.1.2 dev eth0
```


