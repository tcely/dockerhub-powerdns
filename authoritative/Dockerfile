FROM alpine:edge
LABEL maintainer="https://keybase.io/tcely"

ENV PAGER=less

RUN apk --update upgrade && \
    apk add less man \
        pdns-backend-bind pdns-backend-sqlite3 \
        pdns-backend-mysql pdns-backend-pgsql \
        pdns-backend-lua pdns-backend-pipe \
        pdns-backend-random pdns-backend-remote \
        pdns-backend-geoip \
        pdns-tools pdns-doc \
        && \
    rm -rf /var/cache/apk/*

ENTRYPOINT ["/usr/sbin/pdns_server"]
CMD ["--help"]
