#-------------------------------------------------------------------------------
# IMAGE
#-------------------------------------------------------------------------------

# Download image
FROM centos:centos7

# Update packages
RUN yum clean all && \
    yum -y update && \
    yum clean all && \
    rm -rf /var/cache/yum

#-------------------------------------------------------------------------------
# LABELS
#-------------------------------------------------------------------------------

LABEL name="btc_bitcoind"
LABEL application="bitcoind"
LABEL author="liv community"
LABEL license="Simplified BSD License"

#-------------------------------------------------------------------------------
# VARIABLES
#-------------------------------------------------------------------------------

# BITCOIND
ENV BITCOIND_VERSION="0.16.2"
ENV BITCOIND_CHECKSUM="71f217e30e98d5ccc1fb574b9499595e87e118e596278fad5507a7b84650859c"

#-------------------------------------------------------------------------------
# EXPOSE
#-------------------------------------------------------------------------------

EXPOSE 8333/tcp

#-------------------------------------------------------------------------------
# DEPENDENCIES
#-------------------------------------------------------------------------------

# Install dependencies
RUN yum -y install tar && \
    yum -y install wget && \
    yum clean all && \
    rm -rf /var/cache/yum

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

# Download, verify, unarchive and delete 'bitcoind' archive
RUN wget -q -O /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz \
    https://bitcoin.org/bin/bitcoin-core-${BITCOIND_VERSION}/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz && \
    echo "${BITCOIND_CHECKSUM}  /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz" > /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.checksum && \
    sha256sum -c /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.checksum || exit 1 && \
    tar xzf /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.tar.gz -C /app/bitcoind/ && \
    rm -f /tmp/bitcoin-${BITCOIND_VERSION}-x86_64-linux-gnu.*

# Create 'current' symlink
RUN ln -s /app/bitcoind/bitcoin-${BITCOIND_VERSION} /app/bitcoind/current

# Ensure propper POSIX owner and group
RUN chown -R root:root /app/bitcoind/

# Create configuration directory
RUN install --directory --owner=root --group=root --mode=0755 /app/bitcoind/current/conf/

# Copy 'bitcoin.conf'
COPY bitcoin.conf /app/bitcoind/current/conf/

# Run 'bitcoind' as user 'bitcoind'
USER bitcoind
ENTRYPOINT ["/app/bitcoind/current/bin/bitcoind", "-conf=/app/bitcoind/current/conf/bitcoin.conf"]
