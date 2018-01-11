ARG AUTH_VERSION=4.1.0

FROM alpine AS builder

ARG AUTH_VERSION
ARG EXTRA_KEY

RUN apk --update upgrade && \
    apk add ca-certificates && \
    apk add --virtual .build-depends \
      curl gnupg \
      file g++ make patch \
      boost-dev libressl-dev libsodium-dev lua-dev mariadb-dev net-snmp-dev postgresql-dev protobuf-dev sqlite-dev && \
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
	curl -RL -o patch.txt 'https://gist.github.com/tcely/48cb43d0456d96d7a1d32f10ec046cb0/raw/523e9d24d4f131ea8c083114598d22a017666729/pdns-4.1.0-mplexer-timeh.patch.txt' && \
        patch -p 1 -i patch.txt && rm -f patch.txt && \
        ./configure --sysconfdir=/etc/pdns --mandir=/usr/share/man \
            --enable-libsodium --with-sqlite3 --enable-tools \
            --with-modules='bind gsqlite3' \
            --with-dynmodules='gmysql gpgsql lua pipe random remote' && \
        make -j 2 && \
        make install \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine

RUN apk --update upgrade && \
    apk add ca-certificates less man \
        boost-program_options libressl libsodium lua net-snmp protobuf sqlite-libs && \
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

EXPOSE 53/tcp 53/udp
ENTRYPOINT ["/usr/local/sbin/pdns_server"]
CMD ["--help"]
