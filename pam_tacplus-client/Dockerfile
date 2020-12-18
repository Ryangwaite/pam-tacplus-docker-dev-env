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
    pamtester \
    vim

# Need to see syslog messages from PAM-tacplus. With this installed they will
# appear in /var/log/auth.log
RUN apt-get update && apt-get install -y rsyslog

# rsyslogd will log
#   rsyslogd: imklog: cannot open kernel log (/proc/kmsg): Operation not permitted.
# if the default rsyslog.conf is used. Comment out the cuplrit.
# (Borrowed from https://stackoverflow.com/a/60265997)
RUN sed -i '/imklog/s/^/#/' /etc/rsyslog.conf

# Create directory to mount the pam_tacplus git repo
RUN mkdir -p /root/src/pam_tacplus

WORKDIR /root/src/pam_tacplus

# Use rsyslogd to keep the container running so that a bash terminal can be
# attached via "docker-compose exec ..."
ENTRYPOINT [ "/usr/sbin/rsyslogd", "-n" ]