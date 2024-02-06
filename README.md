pages-images
============

A series of common container images used in the cloud.gov Pages system.

## Why this project

Pages relies on Concourse CI and CloudFoundry Apps to deliver and serve the platform's different components. This is a centralized repository to hold all of the commonly used images which are hardened and scanned using the cloud.gov [common-pipelines](https://github.com/cloud-gov/common-pipelines) container hardening CI framework.

## How this works

Create a directory in the root of the repository for the name of the image and then a subdirectory for the images's version. To create a Node v20 image, create the directories `./node/v20` and then add the `Dockerfile` and any other supporting scripts under the image version directory.

All Dockerfiles will take the `ARG base_image` argument which is a hardened Ubuntu Jammy image based the [common-pipelines OCI Pipeline](https://github.com/cloud-gov/common-pipelines/blob/main/container/README.md).

## Current Images

- `dind`: "Docker in Docker" is used to run a CI tasks that leverage docker compose to run linting, compilation, tests, etc. of the source code.

- `node`: The base Node.js image used by the Pages API

## Contributing

See [CONTRIBUTING](CONTRIBUTING.md) for additional information.

## Public domain

This project is in the worldwide [public domain](LICENSE.md). As stated in [CONTRIBUTING](CONTRIBUTING.md):

> This project is in the public domain within the United States, and copyright and related rights in the work worldwide are waived through the [CC0 1.0 Universal public domain dedication](https://creativecommons.org/publicdomain/zero/1.0/).
>
> All contributions to this project will be released under the CC0 dedication. By submitting a pull request, you are agreeing to comply with this waiver of copyright interest.
