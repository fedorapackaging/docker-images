Distrobuilder: build LXC container images
=========================================

# References
* [This README](https://github.com/fedorapackaging/docker-images/blob/master/distrobuilder)
  is part of
  [the Docker images for Fedora packaging project](https://github.com/fedorapackaging/docker-images)
* [Distrobuilder on GitHub](https://github.com/lxc/distrobuilder)
* [Using distrobuilder to create container images for LXC and LXD](https://blog.simos.info/using-distrobuilder-to-create-container-images-for-lxc-and-lxd/)
* [Official templates to generate LXC images](https://github.com/lxc/lxc-ci/tree/master/images)
  + [Official LXC Fedora YAML template](https://github.com/lxc/lxc-ci/blob/master/images/fedora.yaml)

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
$ wget https://github.com/lxc/lxc-ci/raw/master/images/fedora.yaml -O $HOME/container-images/fedora/fedora-30.yaml
$ pushd $HOME/container-images/fedora/
$ cp $HOME/go/src/github.com/lxc/distrobuilder/doc/examples/fedora fedora-distrobuilder.yaml
$ distrobuilder build-lxc fedora-30.yaml
/tmp/fedora-30-x86_64/Fedora-Container-Base-30-20190427.n.0.x86_64.tar.xz: 100% (14.65MB/s)
Fedora Modular 30 - x86_64                   6.7 MB/s | 2.1 MB     00:00    
Fedora Modular 30 - x86_64 - Updates         9.8 MB/s | 2.9 MB     00:00    
Fedora 30 - x86_64 - Updates                 13 MB/s | 8.1 MB     00:00    
Fedora 30 - x86_64                           24 MB/s |  54 MB     00:02    
Metadata cache created.
Last metadata expiration check: 0:00:19 ago on Wed 08 May 2019 09:05:51 PM UTC.
Dependencies resolved.
...
Complete!
33 files removed
Created symlink /etc/systemd/system/multi-user.target.wants/systemd-networkd.service → /usr/lib/systemd/system/systemd-networkd.service.
Created symlink /etc/systemd/system/sockets.target.wants/systemd-networkd.socket → /usr/lib/systemd/system/systemd-networkd.socket.
Created symlink /etc/systemd/system/network-online.target.wants/systemd-networkd-wait-online.service → /usr/lib/systemd/system/systemd-networkd-wait-online.service.
Created symlink /etc/systemd/system/multi-user.target.wants/systemd-resolved.service → /usr/lib/systemd/system/systemd-resolved.service.
$ ls -lh
total 71M
-rw-r--r-- 1 root root 2.1K May  8 23:02 fedora-30.yaml
-rw-r--r-- 1 root root 1.2K May  7 01:09 fedora-distrobuilder-30.yaml
-rw-r--r-- 1 root root  500 May  8 23:07 meta.tar.xz
-rw-r--r-- 1 root root  71M May  8 23:07 rootfs.tar.xz
$ popd
```

* Copy the image to the Proxmox template directory:
```bash
$ export TODAY_DATE=$(date "+%Y%m%d")
$ cp rootfs.tar.xz /vz/template/cache/fedora-30-default_${TODAY_DATE}_amd64.tar.xz
```

# Create a LXC container on a Proxmox
```bash
$ pct create 30 local:vztmpl/fedora-30-default_${TODAY_DATE}_amd64.tar.xz --arch amd64 --cores 8 --hostname fedf30.myowndomain.org --memory 65536 --swap 65536 --net0 name=eth0,bridge=vmbr0,firewall=1,gw=ip.of.my.gw,ip=ip.for.my.vm/32,type=veth --onboot 1 --ostype unmanaged
$ pct resize 30 rootfs 50G
```

* Bootstrap the network
```bash
$ pct start 30
$ pct enter 30
$ hostnamectl set-hostname fedf30.myowndomain.org
$ echo "fedf30.myowndomain.org" > /etc/hostname
$ cat > /etc/sysconfig/network << _EOF
NETWORKING=yes
HOSTNAME=fedf30.myowndomain.org
_EOF
$ ip addr add ip.for.my.vm/24 dev eth0
$ ip link set eth0 up
$ ip route add default via 10.30.1.2 dev eth0
```


