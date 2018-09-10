FROM alpine:edge
LABEL maintainer="https://keybase.io/tcely"

ENV PAGER=less

RUN apk --update upgrade && \
    apk add less man \
        dnsdist-luajit dnsdist-doc \
        && \
    rm -rf /var/cache/apk/*

ENTRYPOINT ["/usr/bin/dnsdist"]
CMD ["--help"]
