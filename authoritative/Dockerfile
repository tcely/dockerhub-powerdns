ARG LUA_VERSION=5.1

FROM alpine AS builder

ARG AUTH_VERSION
ARG LUA_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl jq && \
    apk add --virtual .build-depends \
      file gnupg g++ make \
      boost-dev openssl-dev libsodium-dev lmdb-dev lua${LUA_VERSION}-dev net-snmp-dev protobuf-dev \
      curl-dev mariadb-dev postgresql-dev sqlite-dev geoip-dev yaml-cpp-dev && \
    [ -n "$AUTH_VERSION" ] || { curl -sSL 'https://api.github.com/repos/PowerDNS/pdns/tags?per_page=100&page={1,2,3}' | jq -rs '[.[][]]|map(select(has("name")))|map(select(.name|contains("auth-")))|map(.version=(.name|ltrimstr("auth-")))|map(select(true != (.version|contains("-"))))|map(.version)|"AUTH_VERSION="+.[0]' > /tmp/latest-auth-tag.sh && . /tmp/latest-auth-tag.sh; } && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    curl -RL -O 'https://www.powerdns.com/powerdns-keyblock.asc' && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --import *.asc && \
    curl -RL -O "https://downloads.powerdns.com/releases/pdns-${AUTH_VERSION}.tar.bz2{.asc,.sig,}" && \
    gpg2 --no-options --verbose --keyid-format 0xlong --keyserver-options auto-key-retrieve=true \
        --verify *.sig && \
    rm -rf /root/.gnupg *.asc *.sig && \
    tar -xpf "pdns-${AUTH_VERSION}.tar.bz2" && \
    rm -f "pdns-${AUTH_VERSION}.tar.bz2" && \
    ( \
        cd "pdns-${AUTH_VERSION}" && \
        ./configure --sysconfdir=/etc/pdns --mandir=/usr/share/man \
            --enable-libsodium --with-sqlite3 --enable-tools \
            --with-modules='bind gsqlite3' \
            --with-dynmodules='geoip gmysql gpgsql lmdb lua2 pipe remote' && \
        make -j 2 && \
        make install-strip \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine
LABEL maintainer="https://keybase.io/tcely"

ARG LUA_VERSION

RUN apk --update upgrade && \
    apk add ca-certificates curl less mandoc \
        boost-program_options boost-serialization \
        openssl libsodium lmdb lua${LUA_VERSION}-libs net-snmp protobuf \
        mariadb-connector-c libpq sqlite-libs \
        geoip yaml-cpp && \
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

RUN install -v -d -m 00770 -o root -g pdns /var/run/pdns && ls -l /var/run/

RUN cp -p /etc/pdns/pdns.conf-dist /etc/pdns/pdns.conf && \
    /usr/local/sbin/pdns_server --version || [ $? -eq 99 ]

ENTRYPOINT ["/usr/local/sbin/pdns_server"]
CMD ["--help"]
