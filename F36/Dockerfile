#
# https://github.com/fedorapackaging/docker-images/tree/master/F36
#
FROM fedora:36

LABEL authors="Denis Arnaud <denis.arnaud_github at m4x dot org>"

#
ENV HOME /home/build

# Update of the OS
RUN dnf -y clean all
RUN dnf -y update || echo "Issue with RPM DB, that's fine"

# Basic, convenient
RUN dnf -y install less htop net-tools which sudo keychain man wget vim || echo "Issue with installing less. That's fine"
ADD resources/bashrc /root/.bashrc

# Fedora/CentOS/RedHat packaging
RUN dnf -y install fedora-packager keyutils rpmconf dnf-utils git-all bash-completion Lmod make || echo "Issue with installing fedora-packager. That's fine"

# Specific to C++-based packages (with Python bindings)
RUN dnf -y install gcc-c++ boost-devel cmake python-devel python3-devel bzip2-devel m4 python2-numpy python3-numpy mpich-devel openmpi-devel || echo "Issue with installing boost-devel. That's fine"

# Documentation tools
RUN dnf -y install "tex(latex)" || echo "Issue with installing Latex. That's fine"
RUN dnf -y install doxygen ghostscript || echo "Issue with installing documentation tools. That's fine"

# SOCI, ZeroMQ
RUN dnf -y install zeromq-devel czmq-devel soci-mysql-devel soci-sqlite3-devel readline-devel || echo "Issue with installing zeromq. That's fine"

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

