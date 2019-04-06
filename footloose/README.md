Footloose: Container Machines, Containers as Virtual Machines (VM)
==================================================================

# References
* [This README](https://github.com/fedorapackaging/docker-images/blob/master/footloose)
  is part of
  [the Docker images for Fedora packaging project](https://github.com/fedorapackaging/docker-images)
* Most of the content comes directly from the
  [Footloose open source project, by Weaveworks](https://github.com/weaveworks/footloose),
  full credit to them for their great work!

# Installation

## Linux
```bash
$ curl -Lo footloose https://github.com/weaveworks/footloose/releases/download/0.3.0/footloose-0.3.0-linux-x86_64
$ chmod +x footloose
$ sudo mv footloose /usr/local/bin/
```

## MacOS
```bash
$ brew tap weaveworks/tap
$ brew install weaveworks/tap/footloose
```

## Configuration

### `footloose.yaml`
`footloose` config create creates a `footloose.yaml` configuration file
that is then used by subsequent commands such as `create`, `delete`
or `ssh`. If desired, the configuration file can be named differently
and supplied with the `-c`, `--config` option.
```bash
$ footloose config create --replicas 3
$ cat footloose.yaml
```
```yaml
cluster:
  name: cluster
  privateKey: cluster-key
machines:
- count: 3
  spec:
    image: quay.io/footloose/centos7
    name: node%d
    portMappings:
    - containerPort: 22
```
This configuration can naturally be edited by hand.
The full list of available parameters are in the reference documentation.

## Under the hood
Under the hood, Container Machines are just containers.
They can be inspected with docker:
```bash
$ docker ps
CONTAINER ID    IMAGE                        COMMAND         NAMES
04c27967f76e    quay.io/footloose/centos7    "/sbin/init"    cluster-node2
1665288855f6    quay.io/footloose/centos7    "/sbin/init"    cluster-node1
5134f80b733e    quay.io/footloose/centos7    "/sbin/init"    cluster-node0
```

The container names are derived from `cluster.name`
and `cluster.machines[].name`.

They run systemd as PID 1, it is even possible to inspect
the boot messages:
```bash
$ docker logs cluster-node1
systemd 219 running in system mode.
Detected virtualization docker.
Detected architecture x86-64.

Welcome to CentOS Linux 7 (Core)!

Set hostname to <1665288855f6>.
Initializing machine ID from random generator.
Failed to install release agent, ignoring: File exists
[  OK  ] Created slice Root Slice.
[  OK  ] Created slice System Slice.
[  OK  ] Reached target Slices.
[  OK  ] Listening on Journal Socket.
[  OK  ] Reached target Local File Systems.
         Starting Create Volatile Files and Directories...
[  OK  ] Listening on Delayed Shutdown Socket.
[  OK  ] Reached target Swap.
[  OK  ] Reached target Paths.
         Starting Journal Service...
[  OK  ] Started Create Volatile Files and Directories.
[  OK  ] Started Journal Service.
[  OK  ] Reached target System Initialization.
[  OK  ] Started Daily Cleanup of Temporary Directories.
[  OK  ] Reached target Timers.
[  OK  ] Listening on D-Bus System Message Bus Socket.
[  OK  ] Reached target Sockets.
[  OK  ] Reached target Basic System.
         Starting OpenSSH Server Key Generation...
         Starting Cleanup of Temporary Directories...
[  OK  ] Started Cleanup of Temporary Directories.
[  OK  ] Started OpenSSH Server Key Generation.
         Starting OpenSSH server daemon...
[  OK  ] Started OpenSSH server daemon.
[  OK  ] Reached target Multi-User System.
```

# Usage

* Create a `footloose.yaml` configuration file.
  Instruct we want to create 3 machines.
```bash
$ footloose config create --replicas 3
```

* Start the cluster:
```bash
$ footloose create
INFO[0000] Pulling image: quay.io/footloose/centos7 ...
INFO[0007] Creating machine: cluster-node0 ...
INFO[0008] Creating machine: cluster-node1 ...
INFO[0008] Creating machine: cluster-node2 ...
```

* SSH into a machine with:
```bash
$ footloose ssh root@node1
[root@1665288855f6 ~]# ps fx
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 /sbin/init
   23 ?        Ss     0:00 /usr/lib/systemd/systemd-journald
   58 ?        Ss     0:00 /usr/sbin/sshd -D
   59 ?        Ss     0:00  \_ sshd: root@pts/1
   63 pts/1    Ss     0:00      \_ -bash
   82 pts/1    R+     0:00          \_ ps fx
   62 ?        Ss     0:00 /usr/lib/systemd/systemd-logind
```

* Choosing the OS image to run
  `footloose` will default to running a Centos 7 container image.
  The `--image` argument of config create can be used to configure
  the OS image. Valid OS images are:
```txt
quay.io/footloose/centos7
quay.io/footloose/fedora29
quay.io/footloose/ubuntu16.04
quay.io/footloose/ubuntu18.04
quay.io/footloose/amazonlinux2
quay.io/footloose/debian10
```

* For example:
```bash
$ footloose config create --replicas 3 --image quay.io/footloose/fedora29
```

## Running dockerd in Container Machines
To run `dockerd` inside a docker container, two things are needed:
1. Run the container as privileged (we could probably do better!
   expose capabilities instead!).
2. Mount `/var/lib/docker` as volume, here an anonymous volume.
   This is because of a limitations of what you can do with
   the overlay systems docker is setup to use.
```yaml
cluster:
  name: cluster
  privateKey: cluster-key
machines:
- count: 1
  spec:
    image: quay.io/footloose/centos7
    name: node%d
    portMappings:
    - containerPort: 22
    privileged: true
    volumes:
    - type: volume
      destination: /var/lib/docker
```

* You can then install and run docker on the machine:
```bash
$ footloose create
$ footloose ssh root@node0
# yum install -y docker iptables
[...]
# systemctl start docker
# docker run busybox echo 'Hello, World!'
Hello, World!
```

## Examples
* [Customize the OS image](https://github.com/weaveworks/footloose/blob/master/examples/fedora29-htop/README.md)
* [Ansible example](https://github.com/weaveworks/footloose/blob/master/examples/ansible/README.md)
* [Run Apache in a footloose machine](https://github.com/weaveworks/footloose/blob/master/examples/apache/README.md)

# Packaging for Fedora/CentOS/RedHat
Packaging with Footloose is still experimental at that stage.

## CentOS 7
* Launch the cluster:
```bash
$ footloose create -c footloose/footloose-centos7.yaml 
INFO[0000] Creating SSH key: cluster-key ...            
INFO[0003] Pulling image: quay.io/footloose/centos7 ... 
INFO[0035] Creating machine: cluster-epel70 ...
```

* Check that the Docker images are running:
```bash
$ docker ps
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS              PORTS                   NAMES
9bfcbc40b2fb        quay.io/footloose/centos7   "/sbin/init"        3 minutes ago       Up 3 minutes        0.0.0.0:32769->22/tcp   cluster-epel70
```

* Connect to the cluster and install packages:
```bash
$ footloose ssh -c footloose/footloose-centos7.yaml root@epel70
[root@epel70 ~]# yum -y install epel-release
[root@epel70 ~]# yum -y install less htop net-tools which sudo man wget vim
[root@epel70 ~]# yum -y install fedora-packager keyutils rpmconf yum-utils git-all bash-completion Lmod
[root@epel70 ~]# rpmdev-setuptree
```

* Delete the cluster:
```bash
$ footloose delete -c footloose/footloose-centos7.yaml 
INFO[0000] Deleting machine: cluster-epel70 ...         
```


