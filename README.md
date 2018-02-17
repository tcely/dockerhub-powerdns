# dockerhub-powerdns
Docker images of PowerDNS software built on Alpine Linux
- https://hub.docker.com/r/tcely/dnsdist/
- https://hub.docker.com/r/tcely/powerdns-recursor/
- https://hub.docker.com/r/tcely/powerdns-server/

## Examples of using these images

* ### As a base for your own `Dockerfile`
  ```dockerfile
    FROM tcely/dnsdist

    COPY dnsdist.conf /etc/dnsdist/dnsdist.conf
    EXPOSE 53/tcp 53/udp

    ENTRYPOINT ["/usr/local/bin/dnsdist", "--uid", "dnsdist", "--gid", "dnsdist"]
    CMD ["--disable-syslog"]
  ```
* ### In your `docker-compose.yml`
  ```yaml
    version: '3'
    services:
      dnsdist:
        image: 'tcely/dnsdist'
        restart: 'unless-stopped'
        tty: true
        stdin_open: true
        command: ["--disable-syslog", "--uid", "dnsdist", "--gid", "dnsdist", "--verbose"]
        volumes:
          - ./dnsdist.conf:/etc/dnsdist/dnsdist.conf:ro
        expose:
          - '53'
          - '53/udp'
        ports:
          - '53:53'
          - '53:53/udp'
      recursor:
        image: 'tcely/powerdns-recursor'
        restart: 'unless-stopped'
        command: ["--disable-syslog=yes", "--log-timestamp=no", "--local-address=0.0.0.0", "--setuid=pdns-recursor", "--setgid=pdns-recursor"]
        volumes:
          - ./pdns-recursor.conf:/etc/pdns-recursor/recursor.conf:ro
        expose:
          - '53'
          - '53/udp'

  ```
