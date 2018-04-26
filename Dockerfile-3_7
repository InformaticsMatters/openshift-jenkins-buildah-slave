# A Dockerfile to build an OpenShift Jenkins Slave Agent.
# It's based on the OpenShift Maven image,
# which includes OpenJDK 1.8 and Maven 3.x.

# The v3.7 digest...
FROM openshift/jenkins-slave-maven-centos7@sha256:4b3f21350171e74fa4ee1240355d0239144d300bb9edc6a3e38cf303b33b7587
MAINTAINER Alan Christie (alanbchristie)

ENV INSTALL_PATH /project-atomic

ENV BUILDAH_VERSION 0.16.0
ENV BUILDAH_SUB_PATH src/github.com/projectatomic/buildah

ENV PODMAN_VERSION 0.4.2
ENV PODMAN_SUB_PATH src/github.com/projectatomic/libpod

ENV SKOPEO_VERSION 0.1.28
ENV SKOPEO_SUB_PATH src/github.com/projectatomic/skopeo

ENV GOPATH ${INSTALL_PATH}

USER root

# Install gcc (required to build buildah)
RUN yum -y group install "Development Tools"

# Install packages required by buildah & podman
RUN yum -y install \
    atomic-registries \
    bats \
    btrfs-progs-devel \
    bzip2 \
    conmon \
    containernetworking-cni \
    device-mapper-devel \
    git \
    glibc-devel \
    glibc-static \
    glib2-devel \
    go \
    go-md2man \
    golang \
    golang-github-cpuguy83-go-md2man \
    gpgme-devel \
    iptables \
    libassuan-devel \
    libgpg-error-devel \
    libseccomp-devel \
    libselinux-devel \
    make \
    ostree-devel \
    pkgconfig \
    runc \
    skopeo-containers

# Get, make and install buildah
WORKDIR ${INSTALL_PATH}
RUN git clone https://github.com/projectatomic/buildah ./${BUILDAH_SUB_PATH}
WORKDIR ${BUILDAH_SUB_PATH}
RUN git checkout tags/v${BUILDAH_VERSION}
RUN make; make install

# Get, make and install podman
WORKDIR ${INSTALL_PATH}
RUN git clone https://github.com/projectatomic/libpod ./${PODMAN_SUB_PATH}
WORKDIR ${PODMAN_SUB_PATH}
RUN git checkout tags/v${PODMAN_VERSION} 2> /dev/null
RUN make install.tools; make BUILDTAGS='seccomp selinux apparmor'; make install

# Get, make and install skopeo
WORKDIR ${INSTALL_PATH}
RUN git clone https://github.com/projectatomic/skopeo ./${SKOPEO_SUB_PATH}
WORKDIR ${SKOPEO_SUB_PATH}
RUN git checkout tags/v${SKOPEO_VERSION} 2> /dev/null
RUN make binary-local; make install

# Some handy labels...
LABEL buildah.version=${BUILDAH_VERSION}
LABEL podman.version=${PODMAN_VERSION}
LABEL scopeo.version=${SKOPEO_VERSION}
LABEL name="Jenkins Buildah Slave Agent"
LABEL author="Alan Christie (alanbchristie)"

# Buildah needs to run as root.
# We're root at this stage of the script, so leave the USER alone.
# Do not return to the underlying user id (1001).
WORKDIR ${HOME}
