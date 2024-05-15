ARG base_image

FROM ${base_image}

# Set to skip prompt during USG audit
ENV DEBIAN_FRONTEND=noninteractive

# https://openresty.org/en/linux-packages.html#ubuntu
RUN apt update -y && apt-get -y install --no-install-recommends \
    wget \
    gnupg \
    ca-certificates \
    lsb-release

RUN wget -O - https://openresty.org/package/pubkey.gpg | \
    gpg --dearmor -o /usr/share/keyrings/openresty.gpg

RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/openresty.gpg] http://openresty.org/package/ubuntu $(lsb_release -sc) main" | \
    tee /etc/apt/sources.list.d/openresty.list > /dev/null

RUN apt-get update -y && apt-get -y --no-install-recommends install openresty

EXPOSE 80
STOPSIGNAL SIGQUIT

CMD ["openresty", "-g", "daemon off;"]
