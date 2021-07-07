# Terraform Getting Started Guide on GCP with Cloud Shell

A Getting Started Tutorial for Terraform and Google Cloud Platform (GCP), using
Google's interactive [Cloud Shell](https://cloud.google.com/shell/).

This tutorial will help you learn [Terraform](https://www.terraform.io/intro/index.html "Introduction to
Terraform"), an open source Infrastructure as Code tool.

This tutorial assumes 
you know basic GCP concepts and terminology.

## Launch the tutorial

To follow this tutorial, you will need a Google Cloud Platform account. If
you do not have a GCP account, [create one
now](https://console.cloud.google.com/freetrial/). This tutorial uses services included in the GCP [free
tier](https://cloud.google.com/free/).

Complete this tutorial in [Google's Cloud Shell](https://console.cloud.google.com/cloudshell/open?cloudshell_image=gcr.io/graphite-cloud-shell-images/terraform:latest&cloudshell_git_repo=https://github.com/hashicorp/terraform-getting-started-gcp-cloud-shell&cloudshell_git_branch=master&cloudshell_working_dir=tutorial/&open_in_editor=./main.tf&cloudshell_tutorial=./cloudshell_tutorial.md).

You will be prompted to trust this image. Answer "Yes".

## Use a custom image

This tutorial works with the image used above. However, the version of Terraform included in that image (`v1.0.1`) may not be the latest version. You can build and use a Docker image with the latest version.

### Build the Docker Image

1. Set up Docker and the gcloud command line utility as described in the "Before you begin" section of the [GCP Container Registry Quickstart](https://cloud.google.com/container-registry/docs/quickstart "Container Registry Quickstart Documentation").
1. Build the image:
    ```sh
    docker build . -t terraform-gcp-gsg:v$(date "+%Y-%m-%d")
    ```
1. Optionally, inspect/test image locally:
    ```sh
    docker run -it --entrypoint /bin/sh terraform-gcp-gsg:v$(date "+%Y-%m-%d")
    ```

### Deploy Docker Image the GCP Image Registry

1. Make sure docker is configured to authenticate with gcloud:
    ```sh
    gcloud auth configure-docker
    ```
1. Tag the image, replacing `[PROJECT-ID]` with your Google Cloud's project ID:
    ```sh
    docker tag terraform-gcp-gsg:v$(date "+%Y-%m-%d") gcr.io/[PROJECT-ID]/terraform-gcp-gsg:v$(date "+%Y-%m-%d")
    ```

### Use the Docker Image with Cloud Shell

1. Update the URL above to use the URL to your new docker image.
1. You'll be prompted to trust this image. Answer "Yes".
