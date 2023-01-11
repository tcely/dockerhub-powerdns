FROM alpine:edge
LABEL maintainer="https://keybase.io/tcely"

ENV PAGER=less

# testing repository is not enabled in the current alpine:edge image
RUN sed -i.orig -e '/community$/{p;s/community$/testing/;}' /etc/apk/repositories

RUN apk --update upgrade && \
    apk add less mandoc \
        dnsdist dnsdist-doc \
        && \
    rm -rf /var/cache/apk/*

ENTRYPOINT ["/usr/bin/dnsdist"]
CMD ["--help"]
