ARG base_image

FROM ${base_image}

RUN yarn add playwright \
    && yarn playwright install-deps \
    && yarn playwright install

RUN rm -rf /var/lib/apt/lists/*
