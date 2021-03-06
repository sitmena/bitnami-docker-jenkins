FROM docker.io/bitnami/minideb:buster
FROM debian:buster
LABEL maintainer "Bitnami <containers@bitnami.com>"

RUN whoami
ENV BITNAMI_PKG_CHMOD="-R g+rwX" \
    HOME="/" \
    PATH="/opt/bitnami/java/bin:/opt/bitnami/git/bin:/opt/bitnami/common/bin:/opt/bitnami/nami/bin:$PATH"

COPY prebuildfs /
# Install required system packages and dependencies
RUN install_packages sudo ca-certificates curl fontconfig fonts-dejavu-extra gzip libc6 libcom-err2 libcurl4 libffi6 libgcc1 libgcrypt20 libgmp10 libgnutls30 libgpg-error0 libgssapi-krb5-2 libhogweed4 libidn2-0 libk5crypto3 libkeyutils1 libkrb5-3 libkrb5support0 libldap-2.4-2 libnettle6 libnghttp2-14 libp11-kit0 libpsl5 librtmp1 libsasl2-2 libssh2-1 libssl1.1 libtasn1-6 libunistring2 openssh-client procps tar unzip zlib1g 
#RUN usermod -aG sudo $USER
RUN /build/bitnami-user.sh
RUN /build/install-nami.sh
RUN bitnami-pkg install java-1.8.275-0 --checksum 81e35af4a12129edc3d2fdd6fd7c4fab2dbdd5fba74eab6fe4576381867a55ab
RUN bitnami-pkg install git-2.29.2-0 --checksum aebe650c294f00bacd383f90db2bbf5033dc32acc500d39fa66367f0118b7c9d
RUN bitnami-pkg install tini-0.19.0-1 --checksum 9b1f1c095944bac88a62c1b63f3bff1bb123aa7ccd371c908c0e5b41cec2528d
RUN bitnami-pkg unpack jenkins-2.249.3-0 --checksum 2b887d4ee16337296da08d4987ef9f8380a4b4f8ef148bbcb303c4a88469c7e8
RUN bitnami-pkg install gosu-1.12.0-2 --checksum 4d858ac600c38af8de454c27b7f65c0074ec3069880cb16d259a6e40a46bbc50
RUN apt-get update && apt-get upgrade -y && \
    rm -r /var/lib/apt/lists /var/cache/apt/archives
RUN mkdir -p /usr/share/jenkins/ref

COPY rootfs /
RUN ln -sf /install-plugins.sh /usr/local/bin/install-plugins.sh
ENV BITNAMI_APP_NAME="jenkins" \
    BITNAMI_IMAGE_VERSION="2.249.3-debian-10-r16" \
    DISABLE_JENKINS_INITIALIZATION="no" \
    JAVA_OPTS="" \
    JENKINS_HOME="/opt/bitnami/jenkins/jenkins_home" \
    JENKINS_PASSWORD="bitnami" \
    JENKINS_USERNAME="user" \
    LD_PRELOAD="/opt/bitnami/common/lib/libnss_wrapper.so" \
    LNAME="jenkins" \
    NAMI_PREFIX="/.nami" \
    NSS_WRAPPER_GROUP="/opt/bitnami/jenkins/nss_group" \
    NSS_WRAPPER_PASSWD="/opt/bitnami/jenkins/nss_passwd" \
    OS_ARCH="amd64" \
    OS_FLAVOUR="debian-10" \
    OS_NAME="linux"

# Install docker
RUN apt update && apt install -y \
    apt-transport-https \
    lsb-release \ 
    gnupg-agent

RUN echo "deb [arch=amd64] https://download.docker.com/linux/debian buster stable" >> /etc/apt/sources.list
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - && apt update 
RUN apt install docker-ce docker-ce-cli containerd.io -y

# Install docker-ccompose
RUN curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
   chmod +x /usr/local/bin/docker-compose

# Install python 3.7
RUN apt update && \
    apt install python3.7 -y && \
    apt install python3-distutils -y

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Make python 3.7 the default
RUN echo "alias python=python3.7" >> ~/.bashrc && \
    export PATH=${PATH}:/usr/bin/python3.7 && \
    /bin/bash -c "source ~/.bashrc"

# Install pip
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python3 get-pip.py
RUN python3 -m pip install --upgrade pip

# Install kubectl
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.7.0/bin/linux/amd64/kubectl
RUN chmod +x ./kubectl 
RUN mv ./kubectl /usr/local/bin/kubectl

# install pulumi
RUN curl -fsSL https://get.pulumi.com | sh

# Add key pair
COPY ./jenkins_keys/ /root/.ssh/

EXPOSE 8080 8443 50000

USER 1001
ENTRYPOINT [ "/app-entrypoint.sh" ]
CMD [ "/run.sh" ]
