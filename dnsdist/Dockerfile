ARG LUA_VERSION=5.1

FROM alpine AS builder

ARG DNSDIST_VERSION
ARG LUA_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl jq && \
    apk add --virtual .build-depends \
      file gnupg g++ make \
      boost-dev openssl-dev libsodium-dev lua${LUA_VERSION}-dev net-snmp-dev protobuf-dev \
      libedit-dev re2-dev && \
    [ -n "$DNSDIST_VERSION" ] || { curl -sSL 'https://api.github.com/repos/PowerDNS/pdns/tags?per_page=100&page={1,2,3}' | jq -rs '[.[][]]|map(select(has("name")))|map(select(.name|contains("dnsdist-")))|map(.version=(.name|ltrimstr("dnsdist-")))|map(select(true != (.version|contains("-"))))|map(.version)|"DNSDIST_VERSION="+.[0]' > /tmp/latest-dnsdist-tag.sh && . /tmp/latest-dnsdist-tag.sh; } && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    curl -RL -O 'https://dnsdist.org/_static/dnsdist-keyblock.asc' && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --import *.asc && \
    curl -RL -O "https://downloads.powerdns.com/releases/dnsdist-${DNSDIST_VERSION}.tar.bz2{.asc,.sig,}" && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --verify *.sig && \
    rm -rf /root/.gnupg *.asc *.sig && \
    tar -xpf "dnsdist-${DNSDIST_VERSION}.tar.bz2" && \
    rm -f "dnsdist-${DNSDIST_VERSION}.tar.bz2" && \
    ( \
        cd "dnsdist-${DNSDIST_VERSION}" && \
        ./configure --sysconfdir=/etc/dnsdist --mandir=/usr/share/man \
            --enable-dnscrypt --enable-dns-over-tls --with-libsodium --with-re2 --with-net-snmp && \
        make -j 2 && \
        make install-strip \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine
LABEL maintainer="https://keybase.io/tcely"

ARG LUA_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl less mandoc \
        openssl libsodium lua${LUA_VERSION}-libs net-snmp protobuf \
        libedit re2 && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S dnsdist && \
    adduser -S -D -G dnsdist dnsdist

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/

RUN /usr/local/bin/dnsdist --version

ENTRYPOINT ["/usr/local/bin/dnsdist"]
CMD ["--help"]
