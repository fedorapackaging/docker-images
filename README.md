Docker Images to Build (Official) Fedora/EPEL Packages
======================================================

# Introduction
[That project](https://github.com/fedorapackaging/docker-images)
produces Fedora/CentOS-based Docker images, hosted on [dedicated
public Docker Hub site](https://hub.docker.com/r/fedorapackaging/builder/).
Those Docker images are intended to ease the maintenance work of official
Fedora/EPEL RPM packagers.

That project takes its inspiration from [Alan Franzoni's own
initiative](http://github.com/alanfranz/docker-rpm-builder).

Every time some changes are committed on the [project's GitHub
repository](https://github.com/fedorapackaging/docker-images),
the [Docker images are automatically
rebuilt](https://hub.docker.com/r/fedorapackaging/builder/builds/)
and pushed onto Docker Hub.

For most of Fedora/EPEL RPM packaging needs, picking the Docker image
corresponding to the targeted Fedora/EPEL release is enough.

When the pre-requisites of the packaging procedure become too heavy, however,
the Docker image may be amended so as to add that pre-requisite procedure.
The preferred way to propose amendment of the Docker image is through
[pull requests on the GitHub
project](https://github.com/fedorapackaging/docker-images/pulls).
Once the pull request has been merged, i.e., once the Dockerfile amendment
has been [committed in
GitHub](https://github.com/fedorapackaging/docker-images/commits/master),
the Docker cloud then rebuilds the corresponding Docker image, which becomes
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
* Docker Hub dashboard: https://hub.docker.com/r/fedorapackaging/builder/

# Using the Pre-Built Fedora/EPEL RPM Packaging Images
``<fedora-or-epel-version>`` may be one of ``rawhide``, ``fedora27``,
``fedora26``, ``epel7`` or ``epel6``
```bash
$ docker pull fedorapackaging/builder:<fedora-or-epel-version>
$ docker run --rm --privileged=true -v ~/.ssh/id_rsa:/home/build/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/home/build/.ssh/id_rsa.pub -it fedorapackaging/builder:<fedora-or-epel-version>
[build@5..0 dev]$ kinit <fas-username>@FEDORAPROJECT.ORG
[build@5..0 dev]$ MYPACKAGE=boost
[build@5..0 dev]$ fedpkg clone $MYPACKAGE
[build@5..0 dev]$ cd $MYPACKAGE
[build@5..0 dev]$ fedpkg sources
[build@5..0 dev]$ # Make some changes on the RPM specification file
[build@5..0 dev]$ vi $MYPACKAGE.spec
[build@5..0 dev]$ # Build locally
[build@5..0 dev]$ fedpkg local
[build@5..0 dev]$ # and/or
[Build@5..0 dev]$ cp -a *.{patch,bz2,so,py} ~/dev/packages/SOURCES
[build@5..0 dev]$ cp -a *.spec ~/dev/packages/SPECS
[build@5..0 dev]$ cdbuild
[build@5..0 dev]$ rpmbuild -ba $MYPACKAGE.spec
[build@5..0 dev]$ # If everything went well, commit your work, update the Fedora repository and build with Koji
[build@5..0 dev]$ cd -
[build@5..0 dev]$ fedpkg clog
[build@5..0 dev]$ fedpkg commit -F clog -p
[build@5..0 dev]$ fedpkg build --nowait
[build@5..0 dev]$ exit
$ docker kill fedorapackaging/builder:<fedora-or-epel-version>
```

# Customize a Fedora/EPEL Packaging Docker Image
The images may be customized, and pushed to Docker Hub:
``<fedora-or-epel-version>`` may be one of ``rawhide``, ``fedora27``,
``fedora26``, ``epel7`` or ``epel6``
```bash
$ mkdir -p ~/dev
$ cd ~/dev
$ git clone https://github.com/fedorapackaging/docker-images.git
$ cd docker-images/<fedora-or-epel-version>
$ vi Dockerfile
$ docker build -t fedorapackaging/<fedora-or-epel-version>:beta \
  --build-arg full_name="<your-full-name>" --build-arg email_address="<your-email-address>" \
  --squash .
$ docker push fedorapackaging/<fedora-or-epel-version>:beta
```

# TODO
For any of the following features, an issue may be open [on GitHub](https://github.com/fedorapackaging/docker-images/issues):
1. Have dedicated Docker images per main development stacks, for instance Java, C++, Python, Ruby
(e.g., ``rawhide-java``, ``epel7-cpp``, ``fedora27-scala``)
2. Add Docker images for EPEL6 (and EPEL5?)
 


