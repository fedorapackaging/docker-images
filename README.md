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
* Start the Docker container featuring the target release
  (``<fedora-or-epel-version>`` may be one of ``rawhide``, ``fedora29``,
  ``fedora28``, ``epel7`` or ``epel6``):
```bash
$ docker pull fedorapackaging/builder:<fedora-or-epel-version>
$ docker run --rm --privileged=true -v ~/.ssh/id_rsa:/home/build/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/home/build/.ssh/id_rsa.pub -it fedorapackaging/builder:<fedora-or-epel-version>
[build@5..0 dev]$ 
```

* (From within the Docker container,) Authenticate with Kerberos
  on the Fedora Accounting System (FAS):
```bash
[build@5..0 dev]$ kinit <fas-username>@FEDORAPROJECT.ORG
```

* Clone a Fedora package (eg,
  [Boost](http://www.boost.org) is used as an example here):
```bash
[build@5..0 dev]$ MYPACKAGE=boost
[build@5..0 dev]$ fedpkg clone $MYPACKAGE
[build@5..0 dev]$ cd $MYPACKAGE
```

* Retrieve the ``source`` files (mainly, source tarballs and patches):
```bash
[build@5..0 dev]$ fedpkg sources
```

* Make some changes on the RPM specification file, potentially
  fix patches or even add new ones:
```bash
[build@5..0 dev]$ vi $MYPACKAGE.spec
[build@5..0 dev]$ git add $MYPACKAGE.spec
```

* Build the resulting packege locally:
```bash
[build@5..0 dev]$ fedpkg local
```

* Another Fedora release (eg, Rawhide here) may also be targeted
  thanks to ``mock``:
```bash
$ fedpkg mockbuild --root fedora-rawhide-x86_64
```

* The build may also be done manually, to allow for an easier
  step-by-step approach:
```bash
[Build@5..0 dev]$ cp -a *.{patch,bz2,so,py} ~/dev/packages/SOURCES
[build@5..0 dev]$ cp *.spec ~/dev/packages/SPECS
[build@5..0 dev]$ cdbuild
[build@5..0 dev]$ rpmbuild -ba $MYPACKAGE.spec
```

* If everything went well, commit your work, update the Fedora repository
  and build with Koji:
```bash
[build@5..0 dev]$ cd -
[build@5..0 dev]$ fedpkg clog
[build@5..0 dev]$ fedpkg commit -F clog -p
[build@5..0 dev]$ fedpkg build --nowait
[build@5..0 dev]$ exit
```

* Delete the (temporary) Docker image:
```bash
$ docker kill fedorapackaging/builder:<fedora-or-epel-version>
```

# Customize a Fedora/EPEL Packaging Docker Image
The images may be customized, and pushed to Docker Hub:
``<fedora-or-epel-version>`` may be one of ``rawhide``, ``fedora29``,
``fedora28``, ``epel7`` or ``epel6``
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
(e.g., ``rawhide-java``, ``epel7-cpp``, ``fedora29-scala``)
2. Add Docker images for EPEL6 (and EPEL5?)
 


