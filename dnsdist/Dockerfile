FROM alpine:3 AS builder

ARG DNSDIST_VERSION

RUN apk upgrade --no-cache && \
    apk add --no-cache \
        ca-certificates curl jq \
        file gnupg g++ make \
        boost-dev openssl-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev \
        libedit-dev re2-dev h2o-dev && \
    [ -n "$DNSDIST_VERSION" ] || { curl -sSL 'https://api.github.com/repos/PowerDNS/pdns/tags?per_page=100&page={1,2,3}' | jq -rs '[.[][]]|map(select(has("name")))|map(select(.name|contains("dnsdist-")))|map(.version=(.name|ltrimstr("dnsdist-")))|map(select(true != (.version|contains("-"))))|map(.version)|"DNSDIST_VERSION="+.[0]' > /tmp/latest-dnsdist-tag.sh && . /tmp/latest-dnsdist-tag.sh; } && \
    curl -sSRL -O 'https://dnsdist.org/_static/dnsdist-keyblock.asc' && \
    gpg --import *.asc && \
    curl -sSRL -O "https://downloads.powerdns.com/releases/dnsdist-${DNSDIST_VERSION}.tar.bz2{.asc,.sig,}" && \
    gpg --verify *.sig && \
    tar -xpf "dnsdist-${DNSDIST_VERSION}.tar.bz2" && \
    cd "dnsdist-${DNSDIST_VERSION}" && \
    ./configure \
        --sysconfdir=/etc/dnsdist \
        --mandir=/usr/share/man \
        --enable-dnscrypt \
        --enable-dns-over-tls \
        --enable-dns-over-https \
        --with-libsodium \
        --with-re2 \
        --with-net-snmp && \
    make -j 2 && \
    make install-strip


FROM alpine:3
LABEL maintainer="https://keybase.io/tcely"

RUN apk upgrade --no-cache && \
    apk add --no-cache \
        ca-certificates curl less mdocml \
        openssl libsodium lua net-snmp protobuf \
        libedit re2 h2o

ENV PAGER less

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/

RUN dnsdist --version

ENTRYPOINT ["dnsdist"]
CMD ["--help"]
