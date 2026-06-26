FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    postfix \
    postfix-mysql \
    libmysqlclient21 \
    mailutils \
    openssl \
    && rm -rf /var/lib/apt/lists/*

# Create mail user
RUN useradd -m -u 1001 -s /sbin/nologin vmail && \
    mkdir -p /var/mail/vmail && \
    chown -R vmail:vmail /var/mail/vmail

# Create required directories
RUN mkdir -p /etc/postfix/certs

EXPOSE 25 465

CMD ["/usr/sbin/postfix", "start-fg"]
