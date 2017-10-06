
FROM centos:7
MAINTAINER Denis Arnaud <denis.arnaud_fedora@m4x.org>

# Maintainer details (full name and email address)
ARG full_name
ARG email_address

#
ENV HOME /home/build
ENV FULL_NAME $full_name
ENV EMAIL_ADDRESS $email_address

# Add EPEL repositories
RUN yum -y install epel-release

# Install yum
RUN yum -y clean all
#RUN yum -y install dnf

# Update the OS
#RUN yum -y clean all
RUN yum -y upgrade

# Basic, convenient
RUN yum -y install less htop net-tools which sudo man
ADD resources/bashrc /root/
RUN cat /root/bashrc >> /root/.bashrc && rm -f /root/bashrc

# Fedora/CentOS/RedHat packaging
RUN yum -y install fedora-packager keyutils rpmconf yum-utils git-all bash-completion Lmod

# Specific to C++-based packages (with Python bindings)
RUN yum -y install gcc-c++ boost-devel cmake python-devel python3-devel bzip2-devel m4 python3-numpy mpich-devel openmpi-devel

# Documentation tools
RUN yum -y install "tex(latex)"
RUN yum -y install doxygen ghostscript

# SOCI, ZeroMQ
RUN yum -y install zeromq-devel czmq-devel soci-mysql-devel soci-sqlite3-devel readline-devel

# Create the 'build' user (for the package maintainer)
RUN adduser -m -G mock build
RUN echo "build ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/build && \
    chmod 0440 /etc/sudoers.d/build
# Copy the SSH keys
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh
RUN ssh-keyscan pkgs.fedoraproject.org > $HOME/.ssh/known_hosts
ADD resources/ssh-config $HOME/.ssh/config
RUN chmod 600 $HOME/.ssh/config $HOME/.ssh/known_hosts

# Set up the packaging environment
ADD resources/rpmmacros resources/bashrc resources/gitconfig $HOME/
RUN cat $HOME/bashrc >> $HOME/.bashrc && rm -f $HOME/bashrc && \
    chown -R build.build $HOME
RUN echo "[user]" > $HOME/.gitconfig && \
    echo "    name = $FULL_NAME" >> $HOME/.gitconfig && \
    echo "    email = $EMAIL_ADDRESS" >> $HOME/.gitconfig && \
    cat $HOME/gitconfig >> $HOME/.gitconfig && rm -f $HOME/gitconfig && \
    chown -R build.build $HOME
RUN echo "%packager               $FULL_NAME $EMAIL_ADDRESS" > $HOME/.rpmmacros && \
    echo "%_gpg_name              $FULL_NAME $EMAIL_ADDRESS" >> $HOME/.rpmmacros && \
    cat $HOME/rpmmacros >> $HOME/.rpmmacros && rm -f $HOME/rpmmacros && \
    chown -R build.build $HOME

# Switch to the 'build' user
WORKDIR $HOME

USER build
RUN rpmdev-setuptree

# Check out the Fedora review helper project
RUN mkdir -p $HOME/dev/fedora_packaging
WORKDIR $HOME/dev
RUN git clone https://github.com/fedorapackaging/fedorareviews.git
WORKDIR $HOME/dev/fedora_packaging

#
CMD ["/bin/bash"]

