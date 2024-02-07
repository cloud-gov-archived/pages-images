pages-images
============

A series of common container images used in the cloud.gov Pages system.

## Why this project

Pages relies on Concourse CI and CloudFoundry Apps to deliver and serve the platform's different components. This is a centralized repository to hold all of the commonly used images which are hardened and scanned using the cloud.gov [common-pipelines](https://github.com/cloud-gov/common-pipelines) container hardening CI framework.

## How this works

Create a directory in the root of the repository for the name of the image and then a subdirectory for the images's version. To create a Node v20 image, create the directories `./node/v20` and then add the `Dockerfile` and any other supporting scripts under the image version directory.

All Dockerfiles will take the `ARG base_image` argument which is a hardened Ubuntu Jammy image based the [common-pipelines OCI Pipeline](https://github.com/cloud-gov/common-pipelines/blob/main/container/README.md).

### Registering a new image

To add a new image to `pages-images` you will also need to create a new ECR repository for the common pipeline image put step to use. Add the name of the repository to the [repositories list](https://github.com/cloud-gov/cg-provision/blob/main/terraform/stacks/ecr/stack.tf) in Terraform and make sure the update has been applied.

### Running Locally

To get up and running quickly, you can build an image locally using the Ubuntu Jammy (v22.04) image as your `base_image`.

```sh
# Navigate into the directory of the image and version you want to build
# Use the general Ubuntu Jammy base image for local development
docker buildx build . -t <image-tag> --build-arg base_image=ubuntu:22.04
```

A hardened version of the Ubuntu Jammy image is used as the base image to build the Pages images in CI. To download a hardened image to your local machine, you will need to pull the following credentials from Credhub:
- AWS Access Key
- AWS Secret Key
- Repository Name
- Tag
- AWS Region

You will need to then authenticate and follow AWS's instructions for [private registry authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html). After you have authenticated, you will be able to pull the hardened image.

## CI

The cloud.gov [common-pipelines](https://github.com/cloud-gov/common-pipelines) are set by the [container pipeline](https://github.com/cloud-gov/common-pipelines/blob/main/ci/container/pipeline.yml) in the platform team's CI space and each image pipeline is set in the Pages team CI space based on the [pipeline-pages.yml](https://github.com/cloud-gov/common-pipelines/blob/main/container/pipeline-pages.yml) template. The images must build and complete the USG audit, security scan, CVE check, and software inventory. The image pipelines are available to Pages team members and named after the use and version ie `node-v20`.

## Current Images

- `dind`: "Docker in Docker" is used to run a CI tasks that leverage docker compose to run linting, compilation, tests, etc. of the source code.

- `node`: The base Node.js image used by the Pages API

- `python`: The base Python image

## Contributing

See [CONTRIBUTING](CONTRIBUTING.md) for additional information.

## Public domain

This project is in the worldwide [public domain](LICENSE.md). As stated in [CONTRIBUTING](CONTRIBUTING.md):

> This project is in the public domain within the United States, and copyright and related rights in the work worldwide are waived through the [CC0 1.0 Universal public domain dedication](https://creativecommons.org/publicdomain/zero/1.0/).
>
> All contributions to this project will be released under the CC0 dedication. By submitting a pull request, you are agreeing to comply with this waiver of copyright interest.
