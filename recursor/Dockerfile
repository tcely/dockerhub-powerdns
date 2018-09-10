FROM alpine AS builder

ARG RECURSOR_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl jq && \
    apk add --virtual .build-depends \
      file gnupg g++ make patch \
      boost-dev libressl-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev \
      && \
    [ -n "$RECURSOR_VERSION" ] || { curl -sSL 'https://api.github.com/repos/PowerDNS/pdns/tags?per_page=100&page={1,2}' | jq -rs '[.[][]]|map(select(has("name")))|map(select(.name|contains("rec-")))|map(.version=(.name|ltrimstr("rec-")))|map(select(true != (.version|contains("-"))))|max_by(.version)|"RECURSOR_VERSION="+.version' > /tmp/latest-rec-tag.sh && . /tmp/latest-rec-tag.sh; } && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    curl -RL -O 'https://www.powerdns.com/powerdns-keyblock.asc' && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --import *.asc && \
    curl -RL -O "https://downloads.powerdns.com/releases/pdns-recursor-${RECURSOR_VERSION}.tar.bz2{.asc,.sig,}" && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --verify *.sig && \
    rm -rf /root/.gnupg *.asc *.sig && \
    tar -xpf "pdns-recursor-${RECURSOR_VERSION}.tar.bz2" && \
    rm -f "pdns-recursor-${RECURSOR_VERSION}.tar.bz2" && \
    ( \
        cd "pdns-recursor-${RECURSOR_VERSION}" && \
        curl -sSL 'https://github.com/PowerDNS/pdns/commit/1648b8ff39c705fdee526cd73bf2652982b80087.patch' | patch -p 2 && \
        curl -sSL 'https://github.com/PowerDNS/pdns/commit/b141d89b27e52c3a8e76ca79ec5201d001f4fce9.patch' | patch -p 2 && \
        ./configure --sysconfdir=/etc/pdns-recursor --mandir=/usr/share/man \
            && \
        make -j 2 && \
        make install-strip \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine
LABEL maintainer="https://keybase.io/tcely"

RUN apk --update upgrade && \
    apk add ca-certificates curl less man \
        boost libressl libsodium lua net-snmp protobuf \
        && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S pdns-recursor && \
    adduser -S -D -G pdns-recursor pdns-recursor

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/local/sbin /usr/local/sbin/
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/
COPY --from=builder /etc/pdns-recursor /etc/pdns-recursor/

RUN cp -p /etc/pdns-recursor/recursor.conf-dist /etc/pdns-recursor/recursor.conf

ENTRYPOINT ["/usr/local/sbin/pdns_recursor"]
CMD ["--help"]
