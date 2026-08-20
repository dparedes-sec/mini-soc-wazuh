FROM ubuntu:22.04
RUN apt-get update && apt-get install -y openssh-server sudo rsyslog && mkdir /var/run/sshd
RUN useradd -m labuser && echo "labuser:labpass123" | chpasswd
EXPOSE 22
CMD rsyslogd && /usr/sbin/sshd -D