# This docker file is based on docker-cli https://github.com/docker-library/docker/blob/master/25/cli/Dockerfile
# and the docker-dind https://github.com/docker-library/docker/blob/master/25/dind/Dockerfile files
# to create a docker in docker image
ARG base_image

FROM ${base_image}

# Set to skip prompt during USG audit
ENV DEBIAN_FRONTEND=noninteractive


ENV DOCKER_COMPOSE_VERSION 2.29.2
ENV DOCKER_VERSION 27.1.1
ENV DOCKER_BUILDX_VERSION 0.16.2
ENV DOCKER_TLS_CERTDIR=/certs
# https://github.com/docker/docker/tree/master/hack/dind
ENV DIND_COMMIT 65cfcc28ab37cb75e1560e4b4738719c07c6618e

RUN apt-get update && apt-get install -y --no-install-recommends \
	bash \
	wget \
	ca-certificates \
	openssh-client \
	grep

# pre-add a "docker" group for socket usage
RUN set -eux; \
	addgroup -gid 2375 --system docker

RUN set -eux; \
	\
	sysArch="$(uname -m)"; \
	case "$sysArch" in \
	'x86_64') \
	url="https://download.docker.com/linux/static/stable/x86_64/docker-$DOCKER_VERSION.tgz"; \
	;; \
	'aarch64') \
	url="https://download.docker.com/linux/static/stable/aarch64/docker-$DOCKER_VERSION.tgz"; \
	;; \
	*) echo >&2 "error: unsupported 'docker.tgz' architecture ($sysArch)"; exit 1 ;; \
	esac; \
	\
	wget -O 'docker.tgz' "$url"; \
	\
	tar --extract \
	--file docker.tgz \
	--strip-components 1 \
	--directory /usr/local/bin/ \
	--no-same-owner \
	'docker/docker' \
	; \
	rm docker.tgz; \
	\
	docker --version

RUN set -eux; \
	\
	sysArch="$(uname -m)"; \
	case "$sysArch" in \
	'x86_64') \
	url="https://github.com/docker/buildx/releases/download/v$DOCKER_BUILDX_VERSION/buildx-v$DOCKER_BUILDX_VERSION.linux-amd64"; \
	sha256='43e4c928a0be38ab34e206c82957edfdd54f3e7124f1dadd7779591c3acf77ea'; \
	;; \
	'aarch64') \
	url="https://github.com/docker/buildx/releases/download/v$DOCKER_BUILDX_VERSION/buildx-v$DOCKER_BUILDX_VERSION.linux-arm64"; \
	sha256='775f1ab64aa0e5d901dcc6ecf6843ec3261f27476873760711aa362b403f61f3'; \
	;; \
	*) echo >&2 "warning: unsupported 'docker-buildx' architecture ($sysArch); skipping"; exit 0 ;; \
	esac; \
	\
	wget -O 'docker-buildx' "$url"; \
	echo "$sha256 *"'docker-buildx' | sha256sum -c -; \
	\
	plugin='/usr/local/libexec/docker/cli-plugins/docker-buildx'; \
	mkdir -p "$(dirname "$plugin")"; \
	mv -vT 'docker-buildx' "$plugin"; \
	chmod +x "$plugin"; \
	\
	docker buildx version

RUN set -eux; \
	\
	sysArch="$(uname -m)"; \
	case "$sysArch" in \
	'x86_64') \
	url="https://github.com/docker/compose/releases/download/v$DOCKER_COMPOSE_VERSION/docker-compose-linux-x86_64"; \
	sha256='d037bd4937bf18fba67cff4366e084ee125a3e15c25657ee1aeceff8db3672b4'; \
	;; \
	'aarch64') \
	url="https://github.com/docker/compose/releases/download/v$DOCKER_COMPOSE_VERSION/docker-compose-linux-aarch64"; \
	sha256='1caa6c39b9df88dbd8522403d78b786a4151fa4e881c866725f75b5d99500a9d'; \
	;; \
	*) echo >&2 "warning: unsupported 'docker-compose' architecture ($sysArch); skipping"; exit 0 ;; \
	esac; \
	\
	wget -O 'docker-compose' "$url"; \
	echo "$sha256 *"'docker-compose' | sha256sum -c -; \
	\
	plugin='/usr/local/libexec/docker/cli-plugins/docker-compose'; \
	mkdir -p "$(dirname "$plugin")"; \
	mv -vT 'docker-compose' "$plugin"; \
	chmod +x "$plugin"; \
	\
	ln -sv "$plugin" /usr/local/bin/; \
	docker-compose --version; \
	docker compose version

# https://github.com/docker-library/docker/pull/166
#   dockerd-entrypoint.sh uses DOCKER_TLS_CERTDIR for auto-generating TLS certificates
#   docker-entrypoint.sh uses DOCKER_TLS_CERTDIR for auto-setting DOCKER_TLS_VERIFY and DOCKER_CERT_PATH
# (For this to work, at least the "client" subdirectory of this path needs to be shared between the client and server containers via a volume, "docker cp", or other means of data sharing.)

# also, ensure the directory pre-exists and has wide enough permissions for "dockerd-entrypoint.sh" to create subdirectories, even when run in "rootless" mode
RUN mkdir /certs /certs/client && chmod 1777 /certs /certs/client
# (doing both /certs and /certs/client so that if Docker does a "copy-up" into a volume defined on /certs/client, it will "do the right thing" by default in a way that still works for rootless users)

# https://github.com/docker/docker/blob/master/project/PACKAGERS.md#runtime-dependencies
RUN set -eux; \
	apt-get install -y --no-install-recommends \
	btrfs-progs \
	e2fsprogs \
	# e2fsprogs-extra \
	# ip6tables \
	iptables \
	openssl \
	# shadow-uidmap \
	uidmap \
	xfsprogs \
	# xz \
	xz-utils \
	# pigz: https://github.com/moby/moby/pull/35697 (faster gzip implementation)
	pigz \
	zfsutils-linux \
	zfs-dkms \
	zfs-zed \
	;

# set up subuid/subgid so that "--userns-remap=default" works out-of-the-box
RUN set -eux; \
	addgroup --system dockremap; \
	adduser --ingroup dockremap dockremap; \
	echo 'dockremap:165536:65536' >> /etc/subuid; \
	echo 'dockremap:165536:65536' >> /etc/subgid

RUN set -eux; \
	\
	sysArch="$(uname -m)"; \
	case "$sysArch" in \
	'x86_64') \
	url="https://download.docker.com/linux/static/stable/x86_64/docker-$DOCKER_VERSION.tgz"; \
	;; \
	'armhf') \
	url="https://download.docker.com/linux/static/stable/armel/docker-$DOCKER_VERSION.tgz"; \
	;; \
	'armv7') \
	url="https://download.docker.com/linux/static/stable/armhf/docker-$DOCKER_VERSION.tgz"; \
	;; \
	'aarch64') \
	url="https://download.docker.com/linux/static/stable/aarch64/docker-$DOCKER_VERSION.tgz"; \
	;; \
	*) echo >&2 "error: unsupported 'docker.tgz' architecture ($sysArch)"; exit 1 ;; \
	esac; \
	\
	wget -O 'docker.tgz' "$url"; \
	\
	tar --extract \
	--file docker.tgz \
	--strip-components 1 \
	--directory /usr/local/bin/ \
	--no-same-owner \
	# we exclude the CLI binary because we already extracted that over in the "docker:25-cli" image that we're FROM and we don't want to duplicate those bytes again in this layer
	--exclude 'docker/docker' \
	; \
	rm docker.tgz; \
	\
	dockerd --version; \
	containerd --version; \
	ctr --version; \
	runc --version

RUN set -eux; \
	wget -O /usr/local/bin/dind "https://raw.githubusercontent.com/docker/docker/$DIND_COMMIT/hack/dind"; \
	chmod +x /usr/local/bin/dind

COPY modprobe.sh /usr/local/bin/
COPY docker-entrypoint.sh /usr/local/bin/
COPY dockerd-entrypoint.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/dockerd-entrypoint.sh; \
	chmod +x /usr/local/bin/docker-entrypoint.sh; \
	chmod +x /usr/local/bin/modprobe.sh; \
	export PATH="/usr/local/bin/:$PATH";

VOLUME /var/lib/docker
EXPOSE 2375 2376

ENTRYPOINT ["dockerd-entrypoint.sh"]
CMD []
