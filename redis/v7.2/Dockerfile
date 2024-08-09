ARG base_image

FROM ${base_image}

# Set to skip prompt during USG audit
ENV DEBIAN_FRONTEND=noninteractive

ENV REDIS_VERSION=7.2.5

RUN apt-get update -y && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    libssl-dev \
    tcl \
    wget

# download redis
RUN wget "https://download.redis.io/releases/redis-$REDIS_VERSION.tar.gz"

# confirm hash
RUN wget -O hashfile https://raw.githubusercontent.com/redis/redis-hashes/master/README
RUN cat hashfile | grep "redis-$REDIS_VERSION.tar.gz" | awk '{print $4}' > hash
RUN sha256sum redis-$REDIS_VERSION.tar.gz | awk '{print $1}' > ourhash
RUN if [ `cat ourhash` != `cat hash` ]; then exit 1; fi

# install from source, roughly:
# https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-from-source/
RUN tar -xzvf redis-$REDIS_VERSION.tar.gz
RUN cd redis-$REDIS_VERSION && \
    make distclean && \
    make BUILD_TLS=yes && \
    make test && \
    make install

EXPOSE 6379/tcp
ENTRYPOINT [ "redis-server", "--protected-mode", "no"]
