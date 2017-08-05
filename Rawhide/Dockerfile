
FROM fedora:rawhide
MAINTAINER Denis Arnaud <des.arnaud_fedora@m4x.org>

RUN dnf -y clean all
RUN dnf -y update || echo "Issue with RPM DB, that's fine"

CMD ["/bin/bash"]

