FROM alpine:edge
LABEL maintainer="https://keybase.io/tcely"

ENV PAGER=less

RUN apk --update upgrade && \
    apk add less man \
        pdns-recursor pdns-recursor-doc \
        && \
    rm -rf /var/cache/apk/*

ENTRYPOINT ["/usr/sbin/pdns_recursor"]
CMD ["--help"]
