ARG DNSDIST_VERSION=1.2.0
ARG EXTRA_KEY=D6300CABCBF469BBE392E503A208ED4F8AF58446

FROM alpine AS builder

ARG DNSDIST_VERSION
ARG EXTRA_KEY

RUN apk --update upgrade && \
    apk add ca-certificates && \
    apk add --virtual .build-depends \
      curl gnupg \
      file g++ make \
      boost-dev libedit-dev libsodium-dev lua-dev net-snmp-dev protobuf-dev re2-dev && \
    curl -RL -O "https://downloads.powerdns.com/releases/dnsdist-${DNSDIST_VERSION}.tar.bz2{.asc,}" && \
    curl -RL -o SigningKeys 'https://powerdns.com/powerdns-keyblock.asc' && \
    mkdir -v -m 0700 -p /root/.gnupg && \
    gpg2 --no-options --verbose --keyid-format 0xlong --import SigningKeys && rm -f SigningKeys && \
    ( [ -z "${EXTRA_KEY}" ] || gpg2 --no-options --verbose --keyid-format 0xlong --recv-key "${EXTRA_KEY}" ) && \
    gpg2 --no-options --verbose --keyid-format 0xlong --verify *.asc && \
    rm -rf /root/.gnupg *.asc && \
    tar -xpf "dnsdist-${DNSDIST_VERSION}.tar.bz2" && \
    rm -f "dnsdist-${DNSDIST_VERSION}.tar.bz2" && \
    ( \
        cd "dnsdist-${DNSDIST_VERSION}" && \
        ./configure --sysconfdir=/etc/dnsdist --mandir=/usr/share/man \
            --enable-dnscrypt --enable-libsodium --enable-re2 --with-net-snmp && \
        make -j 2 && \
        make install \
    ) && \
    apk del --purge .build-depends && rm -rf /var/cache/apk/*

FROM alpine

RUN apk --update upgrade && \
    apk add ca-certificates \
        boost less libedit libsodium lua man net-snmp protobuf re2 && \
    rm -rf /var/cache/apk/*

ENV PAGER less

RUN addgroup -S dnsdist && \
    adduser -S -D -G dnsdist dnsdist

COPY --from=builder /usr/local/bin /usr/local/bin/
COPY --from=builder /usr/share/man/man1 /usr/share/man/man1/

ENTRYPOINT ["/usr/local/bin/dnsdist"]
CMD ["--help"]
