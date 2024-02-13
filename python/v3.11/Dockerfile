ARG base_image

FROM ${base_image}

ENV PYTHON_VERSION 3.11.3
ENV GPG_KEY A035C8C19219BA821ECEA86B64E628F8D684696D

# Set to skip prompt during USG audit
ENV DEBIAN_FRONTEND=noninteractive

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# runtime dependencies
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
		curl \
        iptables \
		libbluetooth-dev \
        libncurses5-dev \
        libgdbm-dev \
        libnss3-dev \
        libssl-dev \
        libreadline-dev \
        libffi-dev \
        libsqlite3-dev \
        libbz2-dev \
		libpq-dev \
        openssl \
		tk-dev \
		uuid-dev \
        wget \
        zlib1g-dev;

RUN wget -O python.tgz "https://www.python.org/ftp/python/$PYTHON_VERSION/Python-$PYTHON_VERSION.tgz"; \
	mkdir -p /usr/src/python; \
    tar -xf python.tgz -C /usr/src/python; \
    cd /usr/src/python/Python-${PYTHON_VERSION}; \
    ./configure --enable-optimizations; \
    nproc="$(nproc)"; \
	make -j "$nproc"; \
	make install;

# smoke test
RUN python3 --version;

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 24.0
# https://github.com/docker-library/python/issues/365
ENV PYTHON_SETUPTOOLS_VERSION 65.5.1
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/dbf0c85f76fb6e1ab42aa672ffca6f0a675d9ee4/public/get-pip.py
ENV PYTHON_GET_PIP_SHA256 dfe9fd5c28dc98b5ac17979a953ea550cec37ae1b47a5116007395bfacff2ab9

RUN set -eux; \
	\
	wget -O get-pip.py "$PYTHON_GET_PIP_URL"; \
	echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum -c -; \
	\
	export PYTHONDONTWRITEBYTECODE=1; \
	\
	python3 get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
		--no-compile \
		"pip==$PYTHON_PIP_VERSION" \
		"setuptools==$PYTHON_SETUPTOOLS_VERSION" \
	; \
	rm -f get-pip.py; \
	\
	pip --version;

CMD ["python3"]
