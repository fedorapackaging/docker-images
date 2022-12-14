#
# Dockerfile: https://github.com/fedorapackaging/docker-images/tree/master/EPEL9
#
#
# See also:
# Dockerfile: http://github.com/cpp-projects-showcase/docker-images/tree/master/centos9
# Docker Hub/Cloud page: https://cloud.docker.com/u/infrahelpers/repository/docker/infrahelpers/cpppython
#
# Base image:
# * Hub/cloud: 'stream9' on
#   + Docker.io: https://hub.docker.com/r/tgagor/centos-stream 
#   + Quay.io: https://quay.io/repository/centos/centos?tab=tags
# * Dockerfile: https://github.com/tgagor/docker-centos/blob/master/stream9/Dockerfile
#
FROM quay.io/centos/centos:stream9

LABEL authors="Denis Arnaud <denis.arnaud_github at m4x dot org>"

#
ENV HOME /home/build

# Import the Centos GPG key to prevent warnings
RUN rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-Official && \
  rpm --import https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Testing

# Add the repository for CodeReady Linux Builder (CRB)
RUN dnf -y install 'dnf-command(config-manager)'
RUN dnf config-manager --set-enabled crb

# Add EPEL support
RUN dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

# Update of CentOS
RUN dnf -y clean all && \
    dnf -y upgrade

# Basic, convenient
RUN dnf -y install curl-minimal --skip-broken --best --allowerasing
RUN dnf -y install less htop net-tools which sudo man wget vim
ADD resources/bashrc /root/.bashrc

# Fedora/CentOS/RedHat packaging
#RUN dnf -y install fedora-packager
RUN dnf -y install keyutils rpmconf yum-utils git-all bash-completion Lmod

# Specific to C++- and Python-based packages
RUN dnf -y install gcc-c++ boost-devel cmake
RUN dnf -y install python-devel
RUN dnf -y install libicu-devel bzip2-devel xz-devel m4 mpich-devel openmpi-devel

# Documentation tools
RUN dnf -y install "tex(latex)"
RUN dnf -y install doxygen ghostscript

# SOCI, ZeroMQ
RUN dnf -y install zeromq-devel czmq-devel soci-mysql-devel soci-sqlite3-devel readline-devel

# Create the 'build' user (for the package maintainer)
RUN groupadd mock
RUN adduser -m -G mock build
RUN echo "build ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/build && \
    chmod 0440 /etc/sudoers.d/build

# Configure SSH
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh
RUN ssh-keyscan pkgs.fedoraproject.org > $HOME/.ssh/known_hosts
ADD resources/ssh-config $HOME/.ssh/config
RUN chmod 600 $HOME/.ssh/config $HOME/.ssh/known_hosts

# Set up the packaging environment for the `build` user
ADD resources/rpmmacros $HOME/.rpmmacros
ADD resources/bashrc $HOME/.bashrc
ADD resources/gitconfig $HOME/.gitconfig
ADD resources/vimrc $HOME/.vimrc
RUN chmod 640 $HOME/.rpmmacros $HOME/.bashrc $HOME/.gitconfig $HOME/.vimrc
RUN chown -R build.build $HOME

# Switch to the 'build' user
WORKDIR $HOME

USER build
#RUN rpmdev-setuptree

# Check out the Fedora review helper project
RUN mkdir -p $HOME/dev/fedora_packaging
WORKDIR $HOME/dev
RUN git clone https://github.com/fedorapackaging/fedorareviews.git
WORKDIR $HOME/dev/fedora_packaging

# Entry point
CMD ["/bin/bash"]

