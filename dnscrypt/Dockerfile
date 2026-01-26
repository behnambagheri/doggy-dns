# Use an Ubuntu base image
FROM zydou/dnscrypt:latest

# Install necessary packages
#RUN apt-get update && apt-get install -y \
#    dnscrypt-proxy \
#    && apt-get clean \
#    && rm -rf /var/lib/apt/lists/*

# Copy configuration file
COPY forwarding-rules.txt /etc/dnscrypt-proxy/
COPY cloaking-rules.txt /etc/dnscrypt-proxy/
COPY dnscrypt-proxy.toml /etc/dnscrypt-proxy/

# Expose DNS port
EXPOSE 53/udp
EXPOSE 53/tcp
EXPOSE 443/tcp

# Run dnscrypt-proxy
ENTRYPOINT ["dnscrypt-proxy", "-config", "/etc/dnscrypt-proxy/dnscrypt-proxy.toml"]
