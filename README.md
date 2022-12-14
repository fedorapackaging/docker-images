Docker Images to Build (Official) Fedora/EPEL Packages
======================================================

[![Docker Cloud build status](https://img.shields.io/docker/cloud/build/infrahelpers/fedorapackaging)](https://hub.docker.com/repository/docker/infrahelpers/fedorapackaging/general)
[![Docker Repository on Quay](https://quay.io/repository/fedorapackaging/builder/status "Docker Repository on Quay")](https://quay.io/repository/fedorapackaging/builder)

# Introduction
[That project](https://github.com/fedorapackaging/docker-images)
produces Fedora/CentOS-based Docker images, hosted on [dedicated
public Docker Cloud site](https://hub.docker.com/repository/docker/infrahelpers/fedorapackaging).
Those Docker images are intended to ease the maintenance work of official
Fedora/EPEL RPM packagers.

That project initially took its inspiration from [Alan Franzoni's own
initiative](http://github.com/alanfranz/docker-rpm-builder).

Every time some changes are committed on the [project's GitHub
repository](https://github.com/fedorapackaging/docker-images),
the [Docker images are automatically
rebuilt](https://hub.docker.com/repository/docker/infrahelpers/fedorapackaging/timeline)
and pushed onto Docker Cloud.

For most of Fedora/EPEL RPM packaging needs, picking the Docker image
corresponding to the targeted Fedora/EPEL release is enough.

When the pre-requisites of the packaging procedure become too heavy, however,
the Docker image may be amended so as to add that pre-requisite procedure.
The preferred way to propose amendment of the Docker image is through
[pull requests on the GitHub
project](https://github.com/fedorapackaging/docker-images/pulls).
Once the pull request has been merged, _i.e._, once the `Dockerfile`
amendment has been [committed in
GitHub](https://github.com/fedorapackaging/docker-images/commits/master),
Docker Cloud then rebuilds the corresponding Docker image, which becomes
available for every one to use. 

# Pre-Requisites
## Kerberos authentication via keytab file
* Create a keytab file:
```bash
$ mkdir -p ~/.keytab
$ ktutil 
ktutil:  addent -password -p <fas-username>@FEDORAPROJECT.ORG -k 1 -e des-cbc-md5
Password for <fas-username>@FEDORAPROJECT.ORG: 
ktutil:  wkt <fas-username>.keytab
ktutil:  quit
$ mv <fas-username>.keytab ~/.keytab
```
* Check that the Kerberos authentication works:
```bash
$ kinit <fas-username>@FEDORAPROJECT.ORG -k -t ~/.keytab/<fas-username>.keytab 
```

# Images on Docker Hub
* Docker Hub dashboard: https://hub.docker.com/r/infrahelpers/fedorapackaging/

# Using the Pre-Built Fedora/EPEL RPM Packaging Images
* Start the Docker container featuring the target release
  (`<fedora-or-epel-version>` may be one of `rawhide`, `fedora37`,
  `fedora36`, `epel9`, `epel8` or `epel7`):
```bash
$ docker pull infrahelpers/fedorapackaging:<fedora-or-epel-version>
$ docker run --rm --privileged=true -v ~/.ssh/id_rsa:/home/build/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/home/build/.ssh/id_rsa.pub -it infrahelpers/fedorapackaging:<fedora-or-epel-version>
[build@5..0 fedora_packaging]$ 
```

* (From within the Docker container,) Authenticate with Kerberos
  on the [Fedora Accounting System (FAS)](https://admin.fedoraproject.org/accounts/):
```bash
[build@5..0 fedora_packaging]$ kinit <fas-username>@FEDORAPROJECT.ORG
Password for <fas-username>@FEDORAPROJECT.ORG: 
```

* Setup the user names and email addresses as environment variables for
  subsequent settings. They should match as much as possible with
  [Pagure, the Fedora Git repository](https://src.fedoraproject.org/settings#nav-email-tab):
```bash
[build@5..0 fedora_packaging]$ export FULLNAME="Firstname Lastname"
[build@5..0 fedora_packaging]$ export EMAIL="email@example.com"
```

* Setup the user name and email address for Git:
```bash
[build@5..0 fedora_packaging]$ git config --global user.name "$FULLNAME"
[build@5..0 fedora_packaging]$ git config --global user.email "$EMAIL"
```

* Setup the user names and email address for the RPM packaging:
```bash
[build@5..0 fedora_packaging]$ sed -i -e "s/Firstname Lastname/$FULLNAME/g" ~/.rpmmacros
[build@5..0 fedora_packaging]$ sed -i -e "s/email@example.com/$EMAIL/g" ~/.rpmmacros
```

* Clone a Fedora package (eg, [Boost](http://www.boost.org)
  is used as an example here):
```bash
[build@5..0 fedora_packaging]$ MYPACKAGE=boost
[build@5..0 fedora_packaging]$ fedpkg clone $MYPACKAGE
Cloning into 'boost'...
Enter passphrase for key '/home/build/.ssh/id_rsa': 
remote: Counting objects: 1814, done.
remote: Compressing objects: 100% (1515/1515), done.
remote: Total 1814 (delta 964), reused 592 (delta 279)
Receiving objects: 100% (1814/1814), 2.09 MiB | 1.64 MiB/s, done.
Resolving deltas: 100% (964/964), done.
[build@5..0 fedora_packaging]$ cd $MYPACKAGE
```

* Retrieve the `source` files (mainly, source tarballs and patches):
```bash
[build@5..0 boost]$ fedpkg sources
Downloading boost_1_72_0.tar.bz2
################################################### 100.0%
```

* Make some changes on the RPM specification file, potentially
  fix patches or even add new ones:
```bash
[build@5..0 boost]$ vi $MYPACKAGE.spec
[build@5..0 boost]$ git add $MYPACKAGE.spec
```

* Build the resulting packege locally:
```bash
[build@5..0 boost]$ fedpkg local
```

* Another Fedora release (_e.g._, Rawhide here) may also be targeted
  thanks to `mock`:
```bash
[build@5..0 boost]$ fedpkg mockbuild --root fedora-rawhide-x86_64
```

* Launch a Koji build:
```bash
[build@5..0 fedora_packaging]$ cdbuild
[build@5..0 SPECS]$ rpmbuild -bs $MYPACKAGE.spec
[build@5..0 SPECS]$ koji build --arch-override=x86_64 --scratch --nowait rawhide /home/build/dev/packages/SRPM/$MYPACKAGE-$MYVER.src.rpm
```

* The build may also be done manually, to allow for an easier
  step-by-step approach:
```bash
[build@5..0 boost]$ cp -a *.{patch,bz2,so} ~/dev/packages/SOURCES
[build@5..0 boost]$ cp *.spec ~/dev/packages/SPECS
[build@5..0 boost]$ cdbuild
[build@5..0 SPECS]$ rpmbuild -ba $MYPACKAGE.spec
```

* If everything went well, commit your work, update the Fedora repository
  and build with Koji:
```bash
[build@5..0 SPECS]$ cd -
[build@5..0 boost]$ fedpkg clog
[build@5..0 boost]$ fedpkg commit -F clog -p
[build@5..0 boost]$ fedpkg build --nowait
[build@5..0 boost]$ exit
```

* Delete the (temporary) Docker image:
```bash
$ docker kill infrahelpers/fedorapackaging:<fedora-or-epel-version>
```

# Customize a Fedora/EPEL Packaging Docker Image
The images may be customized, and pushed to Docker Hub:
`<fedora-or-epel-version>` may be one of `rawhide`, `fedora31`,
`fedora30`, `epel8`, `epel7` or `epel6`
```bash
$ mkdir -p ~/dev/fedora
$ cd ~/dev/fedora
$ git clone https://github.com/fedorapackaging/docker-images.git
$ cd docker-images
$ vi <fedora-or-epel-version>/Dockerfile
$ docker build -t infrahelpers/fedorapackaging:<fedora-or-epel-version> <fedora-or-epel-version>/
$ docker run --rm --privileged=true -v ~/.ssh/id_rsa:/home/build/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/home/build/.ssh/id_rsa.pub -it infrahelpers/fedorapackaging:<fedora-or-epel-version>
[build@9..d fedora_packaging]$ exit
$ docker push infrahelpers/fedorapackaging:<fedora-or-epel-version>
```

# TODO
For any of the following features, an issue may be open
[on GitHub](https://github.com/fedorapackaging/docker-images/issues):
1. Have dedicated Docker images per main development stacks,
   for instance Java, C++, Python, Ruby (_e.g._, `rawhide-java`, `epel8-cpp`,
   `fedora37-scala`)
2. Automate regular rebuilds (_e.g._, once a day for Rawhide, weekly for Fedora
   and monthly for EPEL)


