Docker Images to Build (Official) Fedora/EPEL Packages
======================================================

# Using Fedora Packaging Docker Images
``<fedora-or-epel-version>`` may be one of ``rawhide``, ``fedora26``, ``epel7``

```bash
$ docker pull fedorapackaging/<fedora-or-epel-version>:latest
$ docker run --rm --privileged=true -it fedorapackaging/<fedora-or-epel-version>
> cdbuild
> rpmbuild -ba <my-package-spec>
> exit
$ docker kill fedorapackaging/<fedora-or-epel-version>
```

# Building a Fedora Packaging Docker Image
```bash
$ docker build -t fedorapackaging/<fedora-or-epel-version>:latest \
  --build-arg full_name="<your-full-name>" --build-arg email_address="<your-email-address>" \
  --build-arg ssh_prv_key="$(cat ~/.ssh/id_rsa)" --build-arg ssh_pub_key="$(cat ~/.ssh/id_rsa.pub)" \
  --squash .
$ docker push fedorapackaging/<fedora-or-epel-version>:latest
```

