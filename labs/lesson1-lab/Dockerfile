FROM fedora

# Install cool software
RUN dnf -y update && \
    dnf -y install procps-ng iproute openssh-server && \
    dnf clean all

CMD ["/usr/sbin/sshd"] 
