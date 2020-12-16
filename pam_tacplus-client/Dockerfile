FROM ubuntu:18.04

# Install packages needed to build pam_tacplus
RUN apt-get update && apt-get install -y \
    libpam-tacplus \
    autoconf \
    build-essential \
    libtool \
    automake \
    libpam-dev \
    libssl-dev

# Install a few other helpful tools
RUN apt-get update && apt-get install -y \
    iputils-ping \
    iproute2 \
    pamtester

# Create directory to mount the pam_tacplus git repo
RUN mkdir -p /root/src/pam_tacplus

WORKDIR /root/src/pam_tacplus

ENTRYPOINT [ "/bin/bash" ]