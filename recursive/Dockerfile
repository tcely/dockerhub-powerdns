ARG RECURSOR_VERSION=4.1.0

FROM alpine AS builder

ARG RECURSOR_VERSION
ARG EXTRA_KEY

RUN apk --update upgrade && \
    apk add ca-certificates && \
    apk add --virtual .build-depends \
      curl gnupg \
      file g++ make \
      boost-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev && \
    curl -RL -O "https://downloads.powerdns.com/releases/pdns-recursor-${RECURSOR_VERSION}.tar.bz2{.asc,}" && \
    curl -RL -o SigningKeys 'https://powerdns.com/powerdns-keyblock.asc' && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    gpg2 --no-options --verbose --keyid-format 0xlong --import SigningKeys && rm -f SigningKeys && \
    ( [ -z "${EXTRA_KEY}" ] || gpg2 --no-options --verbose --keyid-format 0xlong --recv-key "${EXTRA_KEY}" ) && \
    gpg2 --no-options --verbose --keyid-format 0xlong --verify *.asc && \
    rm -rf /root/.gnupg *.asc && \
    tar -xpf "pdns-recursor-${RECURSOR_VERSION}.tar.bz2" && \
    rm -f "pdns-recursor-${RECURSOR_VERSION}.tar.bz2" && \
    ( \
        cd "pdns-recursor-${RECURSOR_VERSION}" && \
        ./configure --sysconfdir=/etc/pdns-recursor --mandir=/usr/share/man \
            && \
        make -j 2 && \
        make install \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine

RUN apk --update upgrade && \
    apk add ca-certificates less man \
        boost libsodium lua net-snmp protobuf && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S pdns-recursor && \
    adduser -S -D -G pdns-recursor pdns-recursor

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/local/sbin /usr/local/sbin/
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/
COPY --from=builder /etc/pdns-recursor /etc/pdns-recursor/

RUN cp -p /etc/pdns-recursor/recursor.conf-dist /etc/pdns-recursor/recursor.conf

EXPOSE 53/tcp 53/udp
ENTRYPOINT ["/usr/local/sbin/pdns_recursor"]
CMD ["--help"]
