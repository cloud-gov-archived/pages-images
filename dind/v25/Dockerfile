# This docker file is based on docker-cli https://github.com/docker-library/docker/blob/master/25/cli/Dockerfile
# and the docker-dind https://github.com/docker-library/docker/blob/master/25/dind/Dockerfile files
# to create a docker in docker image
ARG base_image

FROM ${base_image}

# Set to skip prompt during USG audit
ENV DEBIAN_FRONTEND=noninteractive


ENV DOCKER_COMPOSE_VERSION 2.24.5
ENV DOCKER_VERSION 25.0.1
ENV DOCKER_BUILDX_VERSION 0.12.1
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
			url='https://download.docker.com/linux/static/stable/x86_64/docker-25.0.1.tgz'; \
			;; \
		'armhf') \
			url='https://download.docker.com/linux/static/stable/armel/docker-25.0.1.tgz'; \
			;; \
		'armv7') \
			url='https://download.docker.com/linux/static/stable/armhf/docker-25.0.1.tgz'; \
			;; \
		'aarch64') \
			url='https://download.docker.com/linux/static/stable/aarch64/docker-25.0.1.tgz'; \
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
			url='https://github.com/docker/buildx/releases/download/v0.12.1/buildx-v0.12.1.linux-amd64'; \
			sha256='716321df8ca9c82ffe96f37e9f4aa1199d4969795836dbd57ef72d12e3ac5085'; \
			;; \
		'armhf') \
			url='https://github.com/docker/buildx/releases/download/v0.12.1/buildx-v0.12.1.linux-arm-v6'; \
			sha256='4b3c78b59c0383ab21327e2902af2ea317e3b85e442b1cd776f0c2a7bbbb2999'; \
			;; \
		'armv7') \
			url='https://github.com/docker/buildx/releases/download/v0.12.1/buildx-v0.12.1.linux-arm-v7'; \
			sha256='fe0a9e7812051a72c47d009bf9373e76e23644cc3291c848ac4a9b6f237e9e75'; \
			;; \
		'aarch64') \
			url='https://github.com/docker/buildx/releases/download/v0.12.1/buildx-v0.12.1.linux-arm64'; \
			sha256='fb72d627f2ee080bba70375c367f4292867821e29ca9a8cf485622f6ede8f436'; \
			;; \
		'ppc64le') \
			url='https://github.com/docker/buildx/releases/download/v0.12.1/buildx-v0.12.1.linux-ppc64le'; \
			sha256='1c649498d727183d82fb243e08856533ea90e8d63bfcd6f8b23b264dbcf7ea7a'; \
			;; \
		'riscv64') \
			url='https://github.com/docker/buildx/releases/download/v0.12.1/buildx-v0.12.1.linux-riscv64'; \
			sha256='152e7b423c1ce888eb0a658321eb8c28cc1d09af01acd5c66eddf8474dddf55c'; \
			;; \
		's390x') \
			url='https://github.com/docker/buildx/releases/download/v0.12.1/buildx-v0.12.1.linux-s390x'; \
			sha256='acee98a9f0550bf2c6c1161cf8067c031ddf0c566c41de7db27847bb72e8ee0b'; \
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
			url='https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-x86_64'; \
			sha256='94355be1d1d395040bbda1490f98d5c7627c30798a7955e1f2a78fda33a4b3e1'; \
			;; \
		'armhf') \
			url='https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-armv6'; \
			sha256='bd402cf44fec9640c29e85184ddd36c9d4094045b296e2833533d53715d0cfc2'; \
			;; \
		'armv7') \
			url='https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-armv7'; \
			sha256='80248b4c2c407b22b24896ff6d6e766be7ca97239c5b8137f47c81b62a1befb4'; \
			;; \
		'aarch64') \
			url='https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-aarch64'; \
			sha256='535e90ff9a7f24384f8a38f9f9ad49125485af7ae5ffda7226d091e5b8126948'; \
			;; \
		'ppc64le') \
			url='https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-ppc64le'; \
			sha256='2143fd5df30c29e22869e5461afac0ca1f2b8d435c544b89fc1d826eb9e52df8'; \
			;; \
		'riscv64') \
			url='https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-riscv64'; \
			sha256='e4534c48b1bdf68d664ccc426800b215d3708f3a492d57e36b0a412f4d229546'; \
			;; \
		's390x') \
			url='https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-s390x'; \
			sha256='7def69d38989d1020c49ab37bb68ab2c29558484e33fc952b5258c84cb1bbda1'; \
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
			url='https://download.docker.com/linux/static/stable/x86_64/docker-25.0.1.tgz'; \
			;; \
		'armhf') \
			url='https://download.docker.com/linux/static/stable/armel/docker-25.0.1.tgz'; \
			;; \
		'armv7') \
			url='https://download.docker.com/linux/static/stable/armhf/docker-25.0.1.tgz'; \
			;; \
		'aarch64') \
			url='https://download.docker.com/linux/static/stable/aarch64/docker-25.0.1.tgz'; \
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
	wget -O /usr/local/bin/dind "https://raw.githubusercontent.com/docker/docker/${DIND_COMMIT}/hack/dind"; \
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
