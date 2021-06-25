# Terraform GSG on GCP with Cloud Shell

A Getting Started Tutorial for Terraform and Google Cloud Platform, using
Google's interactive [Cloud Shell](https://cloud.google.com/shell/).

## Purpose

This tutorial will help you learn how to use
[Terraform](https://www.terraform.io/intro/index.html "Introduction to
Terraform"), an open source "Infrastructure as Code" tool provided by HashiCorp.

Since this tutorial will be using Google Cloud Platform (GCP), it's designed for
those with some experience GCP. While no specialized GCP knowledge is required,
the guide assumes knowledge of basic GCP concepts and terminology.

## Using

You can follow this guide from within Google's Cloud Shell starting with [this link](https://console.cloud.google.com/cloudshell/open?cloudshell_image=gcr.io/graphite-cloud-shell-images/terraform:latest&cloudshell_git_repo=https://github.com/hashicorp/terraform-getting-started-gcp-cloud-shell&cloudshell_git_branch=master&cloudshell_working_dir=tutorial/&open_in_editor=./main.tf&cloudshell_tutorial=./cloudshell_tutorial.md)

You will be prompted to trust this image. Answer "Yes".

To follow the guide, you will need a Google Cloud Platform account.

## Use a Custome Image

This tutorial works fine with the image used above. However, the version of Terraform included in that image may not be the latest version. You can build a Docker image with the latest version instead if you prefer.

### Build the Docker Image

1. Set up docker and the gcloud command line utility as described in the "Before you begin" section of the [GCP Container Registry Quickstart](https://cloud.google.com/container-registry/docs/quickstart "Container Registry Quickstart Documentation").
1. Run: `docker build . -t terraform-gcp-gsg:v$(date "+%Y-%m-%d")`
1. Optionally, inspect/test image - for example:
  `docker run -it --entrypoint /bin/sh terraform-gcp-gsg:v$(date "+%Y-%m-%d")`

### Deploy Docker Image the GCP Image Registry

1. Make sure docker is configured to authenticate with gcloud:
  - `gcloud auth configure-docker`
1. `docker tag terraform-gcp-gsg:v$(date "+%Y-%m-%d") gcr.io/[PROJECT-ID]/terraform-gcp-gsg:v$(date "+%Y-%m-%d")`

### Use the Docker Image with Cloud Shell

1. Update the URL above to use the URL to your new docker image.
1. You'll be prompted to trust this image. Answer "Yes".
