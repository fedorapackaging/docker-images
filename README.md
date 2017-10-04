Docker Images to Build (Official) Fedora/EPEL Packages
======================================================

# Images on Docker Hub
* Docker Hub dashboard: https://hub.docker.com/r/fedorapackaging/builder/

# Using Fedora Packaging Docker Images
``<fedora-or-epel-version>`` may be one of ``rawhide``, ``fedora26``, ``epel7``

```bash
$ docker pull fedorapackaging/<fedora-or-epel-version>:latest
$ docker run --rm --privileged=true -v ~/.ssh/id_rsa:/home/build/.ssh/id_rsa -v ~/.ssh/id_rsa.pub:/home/build/.ssh/id_rsa.pub -it fedorapackaging/<fedora-or-epel-version>
> mkdir -p fedora_packaging
> cd fedora_packaging
> kinit <fas-username>@FEDORAPROJECT.ORG
> fedpkg stdair
> cd stdair
> fedpkg sources
> cp stdair-*.bz2 ~/dev/packages/SOURCES
> cp stdair.spec ~/dev/packages/SPECS
> cdbuild
> rpmbuild -ba stdair.spec
> exit
$ docker kill fedorapackaging/<fedora-or-epel-version>
```

# Building a Fedora Packaging Docker Image
```bash
$ docker build -t fedorapackaging/<fedora-or-epel-version>:latest \
  --build-arg full_name="<your-full-name>" --build-arg email_address="<your-email-address>" \
  --squash .
$ docker push fedorapackaging/<fedora-or-epel-version>:latest
```

