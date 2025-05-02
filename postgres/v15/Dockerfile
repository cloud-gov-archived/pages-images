# adapted from https://github.com/docker-library/postgres/blob/8a631b939a0b4197cb6bef49b50b6c40c80ddf5b/15/bookworm/Dockerfile
# and https://wiki.postgresql.org/wiki/Apt

ARG base_image

FROM golang:1.24 AS gosubuilder
ENV GOSU_VERSION=1.17
WORKDIR /go/src/github.com/tianon
RUN git clone https://github.com/tianon/gosu.git --branch $GOSU_VERSION
WORKDIR /go/src/github.com/tianon/gosu
RUN go mod download
RUN go build

FROM ${base_image}

# Set to skip prompt during USG audit
ENV DEBIAN_FRONTEND=noninteractive

ENV PG_VERSION=15

# explicitly set user/group IDs
RUN set -eux; \
    groupadd -r postgres --gid=999; \
    # https://salsa.debian.org/postgresql/postgresql-common/blob/997d842ee744687d99a2b2d95c1083a2615c79e8/debian/postgresql-common.postinst#L32-35
    useradd -r -g postgres --uid=999 --home-dir=/var/lib/postgresql --shell=/bin/bash postgres; \
    # also create the postgres user's home directory with appropriate permissions
    # see https://github.com/docker-library/postgres/issues/274
    mkdir -p /var/lib/postgresql; \
    chown -R postgres:postgres /var/lib/postgresql

RUN set -ex; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    gnupg \
    ; \
    rm -rf /var/lib/apt/lists/*

# grab gosu for easy step-down from root
COPY --from=gosubuilder /go/src/github.com/tianon/gosu/gosu /usr/local/bin/gosu
RUN set -eux; \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends ca-certificates wget; \
    rm -rf /var/lib/apt/lists/*; \
    dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
    apt-mark auto '.*' > /dev/null; \
    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null; \
    chmod +x /usr/local/bin/gosu; \
    gosu --version; \
    gosu nobody true

# make the "en_US.UTF-8" locale so postgres will be utf-8 enabled by default
RUN set -eux; \
    apt-get update; apt-get install -y --no-install-recommends locales; rm -rf /var/lib/apt/lists/*; \
    localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    libnss-wrapper \
    xz-utils \
    zstd \
    ; \
    rm -rf /var/lib/apt/lists/*

RUN mkdir /docker-entrypoint-initdb.d

# ubuntu postgres version install
RUN apt-get update -y && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    postgresql-common

RUN /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh -y

RUN apt-get install postgresql-$PG_VERSION -y

# make the sample config easier to munge (and "correct by default")
RUN set -eux; \
    dpkg-divert --add --rename --divert "/usr/share/postgresql/postgresql.conf.sample.dpkg" "/usr/share/postgresql/$PG_VERSION/postgresql.conf.sample"; \
    cp -v /usr/share/postgresql/postgresql.conf.sample.dpkg /usr/share/postgresql/postgresql.conf.sample; \
    ln -sv ../postgresql.conf.sample "/usr/share/postgresql/$PG_VERSION/"; \
    sed -ri "s!^#?(listen_addresses)\s*=\s*\S+.*!\1 = '*'!" /usr/share/postgresql/postgresql.conf.sample; \
    grep -F "listen_addresses = '*'" /usr/share/postgresql/postgresql.conf.sample

RUN mkdir -p /var/run/postgresql && chown -R postgres:postgres /var/run/postgresql && chmod 2777 /var/run/postgresql

ENV PGDATA /var/lib/postgresql/data
RUN mkdir -p "$PGDATA" && chown -R postgres:postgres "$PGDATA" && chmod 777 "$PGDATA"
VOLUME /var/lib/postgresql/data

# disable postgres user shell
RUN usermod --shell /bin/false postgres

COPY docker-entrypoint.sh /usr/local/bin/
ENV PATH="${PATH}:/usr/lib/postgresql/$PG_VERSION/bin/"
ENTRYPOINT ["docker-entrypoint.sh"]

STOPSIGNAL SIGINT

EXPOSE 5432
CMD ["postgres"]
