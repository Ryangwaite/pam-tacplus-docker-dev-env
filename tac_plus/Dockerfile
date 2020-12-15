
FROM ubuntu:18.04

# Install packages
RUN apt-get update && apt-get install -y sudo tacacs+

ENTRYPOINT [ "/usr/sbin/tac_plus" ]
CMD ["-g", "-C", "/etc/tacacs+/tac_plus.conf", "-d", "16" ]