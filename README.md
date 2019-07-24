# Terraform GSG on GCP with Cloud Shell

A Getting Started Guide for Terraform and Google Cloud Platform, using Google's interactive [Cloud Shell](https://cloud.google.com/shell/ "Google Cloud Shell homepage").

## Purpose

This guide will help you learn how to use [Terraform](https://www.terraform.io/intro/index.html "Introduction to Terraform"), an open source "Infrastructure as Code" tool provided by Hashicorp.

Since this guide will be using Google Cloud Platform (GCP), it's designed for those with some experience GCP. While no specialized GCP knowledge is required, the guide assumes knowledge of basic GCP concepts and terminology.

## Using

You can follow this guide from within Google's Cloud Shell starting with [this link](https://console.cloud.google.com/cloudshell/open?cloudshell_image=gcr.io/graphite-cloud-shell-images/terraform:0.12&cloudshell_git_repo=https://github.com/robin-norwood/terraform-getting-started-gcp-cloud-shell&cloudshell_git_branch=master&cloudshell_working_dir=tutorial/&open_in_editor=./tutorial/main.tf&cloudshell_tutorial=./cloudshell_tutorial.md)

To follow the guide, you'll need an active Google Cloud Platform account.

## Code

The source code for this guide is hosted in [this GitHub repository](https://github.com "FIXME: Link to GH repo").

## Building

### Build the docker image

1. Set up docker and gcloud as described in the "Before you begin" section of the [GCP Container Registry Quickstart](https://cloud.google.com/container-registry/docs/quickstart "Container Registry Quickstart Documentation").
1. Run: `docker build . -t terraform-gcp-gsg:v$(date "+%Y-%m-%d")`
1. Optionally, inspect/test image - for example:
  `docker run -it --entrypoint /bin/sh terraform-gcp-gsg:v$(date "+%Y-%m-%d")`

### Deploy docker image to the image registry

1. Make sure docker is configured to authenticate with gcloud:
  - `gcloud auth configure-docker`
1. `docker tag terraform-gcp-gsg:v$(date "+%Y-%m-%d") gcr.io/[PROJECT-ID]/terraform-gcp-gsg:v$(date "+%Y-%m-%d")`

### Using the docker image with Cloud Shell

1. Update the URL above to use the URL to your new docker image.
1. You'll be prompted to trust this image. Answer "Yes".

## Known Issues

1. The `modules` section does not work yet.
1. Networking problems tend to leave the Google Cloud Shell in an odd state, sometimes requiring the user to start over from scratch.
1. There are some issues with using the `google_project_services` resource, specifically:
  1. if `oslogin.googleapis.com` isn't included, it gets silently enabled; and shows up as "removed" in subsequent runs.
  1. If it is included, things work fine until you run `terraform destroy`:
    Error: Unable to destroy google_project_services for sturdy-mechanic-247714: Error disabling service "oslogin.googleapis.com" for project "sturdy-mechanic-247714": Error waiting for api to disable: Error code 9, message: [Could not turn off service, as it still has resource s in use.] with failed services [compute.googleapis.com]
