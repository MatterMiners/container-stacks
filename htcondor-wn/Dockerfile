FROM rockylinux:9
LABEL maintainer="Manuel Giffels <giffels@gmail.com>"

# Create links to point to scratch directory
RUN mkdir -p /scratch/etc/condor
RUN mkdir -p /scratch/var/log/condor
RUN mkdir -p /scratch/var/lib/condor
RUN mkdir -p /scratch/etc/cloud
RUN mkdir -p /scratch/var/lock/condor
RUN mkdir -p /scratch/var/run/condor

RUN ln -s /scratch/etc/condor /etc/condor
RUN ln -s /scratch/var/log/condor /var/log/condor
RUN ln -s /scratch/var/lib/condor /var/lib/condor
RUN ln -s /scratch/etc/cloud /etc/cloud
RUN mkdir /run/lock
RUN ln -s /scratch/var/lock/condor /run/lock/condor
RUN ln -s /scratch/var/run/condor /var/run/condor

# Accept SHA1 signature keys for HTCondor Repo signing key
RUN update-crypto-policies --set DEFAULT:SHA1

RUN rpm --import http://research.cs.wisc.edu/htcondor/yum/RPM-GPG-KEY-HTCondor
RUN dnf install -y https://research.cs.wisc.edu/htcondor/repo/24.x/el9/x86_64/release/htcondor-release-24.x-1.el9.noarch.rpm

RUN dnf -y install epel-release && dnf clean all
RUN dnf config-manager --set-enabled crb
RUN dnf -y install condor \
                   python3.11 \
                   python3.11-pip \
                   ansible \
                   apptainer \
                   openssh-server \
                   langpacks-en \
                   glibc-langpack-en \
     && dnf clean all

# Get a newer Git version to use Git Protocol v2
RUN dnf -y install https://packages.endpointdev.com/rhel/9/main/x86_64/endpoint-repo.noarch.rpm \
    && dnf config-manager --disable endpoint \
    && dnf -y install --enablerepo endpoint git \
    && dnf clean all

RUN python3.11 -m pip install --no-cache-dir condor_git_config pexpect

# Add ansible community collection
RUN ansible-galaxy collection install community.general -p /usr/share/ansible/collections

# Add Tini
ENV TINI_VERSION=v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod a+rx /tini

COPY ansible_config /srv/ansible_config
COPY pilot_execute.sh /srv/pilot_execute.sh
RUN chmod a+rx /srv/pilot_execute.sh

# Tell sshd not to chroot inside the container
RUN sed -i '/^UsePrivilegeSeparation/{h;s/.*/UsePrivilegeSeparation no/g};${x;/^$/{s//UsePrivilegeSeparation no/;H};x}' /usr/lib64/condor/condor_ssh_to_job_sshd_config_template

# To avoid problems with tty in user namepspaces (https://bugzilla.mindrot.org/show_bug.cgi?id=2813): suppress tty from /etc/group
RUN sed -i '/tty/d' /etc/group

# Set language settings, ansible otherwise complains
ENV LC_ALL=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8

WORKDIR /scratch
ENTRYPOINT ["/tini", "-s", "--", "/srv/pilot_execute.sh"]
