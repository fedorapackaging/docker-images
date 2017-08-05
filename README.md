Docker Images to Build (Official) Fedora/EPEL Packages
======================================================

# Fedora Rawhide

## Get the Docker image from Docker Hub
```bash
$ docker pull fedorapackaging/rawhide:latest
```

## Build the Docker image
```bash
$ docker build -t fedorapackaging/rawhide:latest --squash .
$ 
```

# Fedora 26

## Get the Docker image from Docker Hub
```bash
$ docker pull fedorapackaging/fedora26:latest
```

## Build the Docker image
```bash
$ docker build -t fedorapackaging/fedora26:latest --squash .
```

# EPEL 7

## Get the Docker image from Docker Hub
```bash
$ docker pull fedorapackaging/epel7:latest
```

## Build the Docker image
```bash
$ docker build -t fedorapackaging/epel7:latest --squash .
```


