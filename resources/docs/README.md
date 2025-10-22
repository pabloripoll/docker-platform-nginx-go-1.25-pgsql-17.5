# Docker Postgres 16

## From

- https://github.com/docker-library/postgres/
- https://github.com/docker-library/postgres/tree/master/17/alpine3.22

## Container Information

Operate System
```bash
$ cat /etc/os-release

NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.22.1
PRETTY_NAME="Alpine Linux v3.22"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
```

Version
```bash
$ postgres --version

postgres (PostgreSQL) 17.5
```

## Doker Image Improvements

```bash
# From repository
#ENTRYPOINT ["docker-entrypoint.sh"]

# Fixed
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

RUN chmod +x /usr/local/bin/docker-entrypoint.sh
```
