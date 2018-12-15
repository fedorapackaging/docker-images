#
# https://github.com/fedorapackaging/docker-images/tree/master/EPEL6
#
FROM centos:6
MAINTAINER Denis Arnaud <denis.arnaud_fedora@m4x.org>

#
ENV HOME /home/build

# Add EPEL repositories
RUN yum -y install epel-release

# Update the OS
RUN yum -y clean all
RUN yum -y upgrade

# Basic, convenient
RUN yum -y install less htop net-tools which sudo man wget vim
ADD resources/bashrc /root/.bashrc

# Fedora/CentOS/RedHat packaging
RUN yum -y install fedora-packager keyutils rpmconf yum-utils git-all bash-completion Lmod

# Specific to C++- and Python-based packages
RUN yum -y install gcc-c++ boost-devel cmake cmake3
RUN yum -y install python-devel numpy python34-devel python34-numpy
RUN yum -y install libicu-devel bzip2-devel m4 mpich-devel openmpi-devel

# Documentation tools
RUN yum -y install "tex(latex)"
RUN yum -y install doxygen ghostscript

# SOCI, ZeroMQ
RUN yum -y install zeromq-devel soci-mysql-devel soci-sqlite3-devel readline-devel

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

