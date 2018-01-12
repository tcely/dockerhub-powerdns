ARG AUTH_VERSION=4.1.0

FROM alpine AS builder

ARG AUTH_VERSION
ARG EXTRA_KEY

ADD pdns-4.1.0-mplexer-timeh.patch.txt patch.txt

RUN apk --update upgrade && \
    apk add ca-certificates curl && \
    apk add --virtual .build-depends \
      file gnupg g++ make patch \
      boost-dev libressl-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev \
      mariadb-dev postgresql-dev sqlite-dev && \
    curl -RL -O "https://downloads.powerdns.com/releases/pdns-${AUTH_VERSION}.tar.bz2{.asc,}" && \
    curl -RL -o SigningKeys 'https://powerdns.com/powerdns-keyblock.asc' && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    gpg2 --no-options --verbose --keyid-format 0xlong --import SigningKeys && rm -f SigningKeys && \
    ( [ -z "${EXTRA_KEY}" ] || gpg2 --no-options --verbose --keyid-format 0xlong --recv-key "${EXTRA_KEY}" ) && \
    gpg2 --no-options --verbose --keyid-format 0xlong --verify *.asc && \
    rm -rf /root/.gnupg *.asc && \
    tar -xpf "pdns-${AUTH_VERSION}.tar.bz2" && \
    rm -f "pdns-${AUTH_VERSION}.tar.bz2" && \
    ( \
        cd "pdns-${AUTH_VERSION}" && \
        patch -p 1 -i ../patch.txt && rm -f ../patch.txt && \
        ./configure --sysconfdir=/etc/pdns --mandir=/usr/share/man \
            --enable-libsodium --with-sqlite3 --enable-tools \
            --with-modules='bind gsqlite3' \
            --with-dynmodules='gmysql gpgsql lua pipe random remote' && \
        make -j 2 && \
        make install-strip \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine

RUN apk --update upgrade && \
    apk add ca-certificates curl less man \
        boost-program_options libressl libsodium lua net-snmp protobuf \
        sqlite-libs && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S pdns && \
    adduser -S -D -G pdns pdns

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/local/sbin /usr/local/sbin/
COPY --from=builder /usr/local/lib/pdns /usr/local/lib/pdns
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/
COPY --from=builder /usr/local/share/doc/pdns /usr/local/share/doc/pdns
COPY --from=builder /etc/pdns /etc/pdns/

RUN cp -p /etc/pdns/pdns.conf-dist /etc/pdns/pdns.conf

ENTRYPOINT ["/usr/local/sbin/pdns_server"]
CMD ["--help"]
