Docker Images to Build (Official) Fedora/EPEL Packages
======================================================

# Using Fedora Packaging Docker Images
``<fedora-or-epel-version>`` may be one of ``rawhide``, ``fedora26``, ``epel7``

```bash
$ docker pull fedorapackaging/<fedora-or-epel-version>:latest
$ docker run --rm -it fedorapackaging/<fedora-or-epel-version>
> cdfedorareview
> rpmbuild -ba <my-package-spec>
> exit
$ docker kill fedorapackaging/<fedora-or-epel-version>
```

# Building a Fedora Packaging Docker Image
```bash
$ docker build -t fedorapackaging/<fedora-or-epel-version>:latest --squash .
$ docker push fedorapackaging/<fedora-or-epel-version>:latest
```

