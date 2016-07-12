##################################################
#                                                #
# Creates a base rhel6  image with OpenSSH       #
#                                                #
##################################################

# Use the centos base image
FROM centos:centos6

MAINTAINER mamurai

# Setup init script
ADD init.sh /usr/local/bin/init.sh
RUN chmod u+x /usr/local/bin/init.sh \
	&& mkdir /root/.ssh \
	&& chmod 700 /root/.ssh
ADD authorized_keys /root/.ssh/authorized_keys
RUN chmod 600  /root/.ssh/authorized_keys


# Update the system
RUN yum -y update \
	&& yum -y install openssh-server openssh openssh-clients unzip coreutils shadow-utils sudo \
	&& yum clean all

#sshd setting & Set root Password
RUN cp -p /etc/ssh/sshd_config /etc/ssh/sshd_config.orig \
	&&  sed -ri 's/^UsePAM yes/#UsePAM yes/' /etc/ssh/sshd_config \
	&& sed -ri 's/#^UsePAM no/UsePAM no/' /etc/ssh/sshd_config \
	&& echo 'root:redhat1!' |chpasswd


EXPOSE 22 80 8080 9990 9999

CMD ["/usr/local/bin/init.sh"]

