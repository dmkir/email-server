FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    dovecot-core \
    dovecot-imapd \
    dovecot-mysql \
    mysql-client \
    openssl \
    && rm -rf /var/lib/apt/lists/*

# Create mail user
RUN useradd -m -u 1001 -s /sbin/nologin vmail && \
    mkdir -p /var/mail/vmail && \
    chown -R vmail:vmail /var/mail/vmail

# Create required directories
RUN mkdir -p /etc/dovecot/certs && \
    mkdir -p /var/run/dovecot

EXPOSE 993

CMD ["/usr/sbin/dovecot", "-F"]
