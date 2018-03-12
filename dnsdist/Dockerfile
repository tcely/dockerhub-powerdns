FROM alpine AS builder

ARG DNSDIST_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl jq && \
    apk add --virtual .build-depends \
      file gnupg g++ make \
      boost-dev libressl-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev \
      libedit-dev re2-dev && \
    [ -n "$DNSDIST_VERSION" ] || { curl -sSL 'https://api.github.com/repos/PowerDNS/pdns/tags?per_page=100' | jq -r '[.[]["name"]|select(contains("dnsdist-"))|ltrimstr("dnsdist-")|select(true != contains("-"))][0]|"DNSDIST_VERSION="+.' > /tmp/latest-dnsdist-tag.sh && . /tmp/latest-dnsdist-tag.sh; } && \
    curl -RL -O "https://downloads.powerdns.com/releases/dnsdist-${DNSDIST_VERSION}.tar.bz2{.asc,}" && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --verify *.asc && \
    rm -rf /root/.gnupg *.asc && \
    tar -xpf "dnsdist-${DNSDIST_VERSION}.tar.bz2" && \
    rm -f "dnsdist-${DNSDIST_VERSION}.tar.bz2" && \
    ( \
        cd "dnsdist-${DNSDIST_VERSION}" && \
        ./configure --sysconfdir=/etc/dnsdist --mandir=/usr/share/man \
            --enable-dnscrypt --enable-libsodium --enable-re2 --with-net-snmp && \
        make -j 2 && \
        make install-strip \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

# No aport for dnsdist yet.
# See https://github.com/alpinelinux/aports/pull/3321
FROM alpine
LABEL maintainer="https://keybase.io/tcely"

RUN apk --update upgrade && \
    apk add ca-certificates curl less man \
        boost libressl libsodium lua net-snmp protobuf \
        libedit re2 && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S dnsdist && \
    adduser -S -D -G dnsdist dnsdist

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/

ENTRYPOINT ["/usr/local/bin/dnsdist"]
CMD ["--help"]
