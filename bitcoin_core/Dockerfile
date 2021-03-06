#-------------------------------------------------------------------------------
# IMAGE
#-------------------------------------------------------------------------------

# Download image
FROM centos:centos8

# Update packages
RUN dnf clean all && \
    dnf -y update && \
    dnf clean all && \
    rm -rf /var/cache/dnf

#-------------------------------------------------------------------------------
# LABELS
#-------------------------------------------------------------------------------

LABEL name="bitcoin_core"
LABEL application="bitcoind"
LABEL author="liv community"
LABEL license="Simplified BSD License"

#-------------------------------------------------------------------------------
# VARIABLES
#-------------------------------------------------------------------------------

# BITCOIND
ENV BITCOIND_VERSION="0.19.0.1"

#-------------------------------------------------------------------------------
# EXPOSE
#-------------------------------------------------------------------------------

# Bitcoin RPC
EXPOSE 8332/tcp

# Bitcoin Network
EXPOSE 8333/tcp

# Bitcoin ZeroMQ
EXPOSE 5555/tcp

#-------------------------------------------------------------------------------
# DEPENDENCIES
#-------------------------------------------------------------------------------

# Install dependencies
RUN dnf -y install tar && \
    dnf -y install wget && \
    dnf clean all && \
    rm -rf /var/cache/dnf

#-------------------------------------------------------------------------------
# PERSISTENCE
#-------------------------------------------------------------------------------

# Create data directory
RUN install --directory --owner=root --group=root --mode=0755 /data/bitcoind/

# Mount external volume
VOLUME /data/bitcoind/

#-------------------------------------------------------------------------------
# BITCOIND
#-------------------------------------------------------------------------------

# Create app directory
RUN install --directory --owner=root --group=root --mode=0755 /app/bitcoind/

# Create 'bitcoind' user and group
RUN useradd -c 'bitcoind' -m -p '!' -u 10000 -s '/bin/bash' bitcoind

# Copy GPG public key and create keyring
COPY bitcoin.pem /root/.gnupg/bitcoin.pem
RUN gpg --no-default-keyring --keyring /root/.gnupg/bitcoin.gpg --fingerprint && \
    gpg --no-default-keyring --keyring /root/.gnupg/bitcoin.gpg --batch --import /root/.gnupg/bitcoin.pem

# Download, verify, unarchive and delete 'bitcoin-core' archive
RUN wget -q -O /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz \
    https://bitcoin.org/bin/bitcoin-core-${BITCOIND_VERSION}/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz && \
    wget -q -O /tmp/SHA256SUMS.asc https://bitcoin.org/bin/bitcoin-core-${BITCOIND_VERSION}/SHA256SUMS.asc && \
    gpgv --keyring /root/.gnupg/bitcoin.gpg /tmp/SHA256SUMS.asc || exit 1 && \
    grep " bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz\$" /tmp/SHA256SUMS.asc > /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.checksum && \
    cd /tmp && sha256sum -c /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.checksum || exit 1 && \
    tar xzf /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz -C /app/bitcoind/ && \
    rm -f /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.*

# Create 'current' symlink
RUN ln -s /app/bitcoind/bitcoin-${BITCOIND_VERSION} /app/bitcoind/current

# Ensure propper POSIX owner and group
RUN chown -R root:root /app/bitcoind/

# Create configuration directory
RUN install --directory --owner=root --group=root --mode=0755 /app/bitcoind/bitcoin-${BITCOIND_VERSION}/conf/

# Copy 'bitcoin.conf'
COPY bitcoin.conf /app/bitcoind/bitcoin-${BITCOIND_VERSION}/conf/

# Run 'bitcoind' as user 'bitcoind'
USER bitcoind
CMD ["/app/bitcoind/current/bin/bitcoind", "-conf=/app/bitcoind/current/conf/bitcoin.conf"]
