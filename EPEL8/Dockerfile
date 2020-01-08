#
# https://github.com/fedorapackaging/docker-images/tree/master/EPEL8
#
FROM centos:8
MAINTAINER Denis Arnaud <denis.arnaud_fedora@m4x.org>

#
ENV HOME /home/build

# Import the Centos-8 GPG key to prevent warnings
RUN rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-Official && \
  rpm --import https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Testing

# Add the repository for PowerTools
RUN dnf -y install 'dnf-command(config-manager)'
RUN dnf config-manager --set-enabled PowerTools

# Add EPEL support
RUN dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# Update of CentOS
RUN dnf -y clean all && \
    dnf -y upgrade

# Basic, convenient
RUN dnf -y install less htop net-tools which sudo man wget curl vim
ADD resources/bashrc /root/.bashrc

# Fedora/CentOS/RedHat packaging
RUN dnf -y install fedora-packager keyutils rpmconf yum-utils git-all bash-completion Lmod

# Specific to C++- and Python-based packages
RUN dnf -y install gcc-c++ boost-devel cmake cmake3
RUN dnf -y install python36-devel python3-numpy
RUN dnf -y install libicu-devel bzip2-devel m4 mpich-devel openmpi-devel

# Documentation tools
RUN dnf -y install "tex(latex)"
RUN dnf -y install doxygen ghostscript

# SOCI, ZeroMQ
RUN dnf -y install zeromq-devel czmq-devel soci-mysql-devel soci-sqlite3-devel readline-devel

# Create the 'build' user (for the package maintainer)
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
RUN rpmdev-setuptree

# Check out the Fedora review helper project
RUN mkdir -p $HOME/dev/fedora_packaging
WORKDIR $HOME/dev
RUN git clone https://github.com/fedorapackaging/fedorareviews.git
WORKDIR $HOME/dev/fedora_packaging

# Entry point
CMD ["/bin/bash"]

