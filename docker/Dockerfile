# Only needed if we want the latest and greatest terraform. Otherwise, just use:
#
# https://console.cloud.google.com/cloudshell/open?cloudshell_image=gcr.io/graphite-cloud-shell-images/terraform:0.12

FROM gcr.io/cloudshell-images/cloudshell:cloud-shell-v20190709

# Install terraform:

RUN curl -o /tmp/terraform.zip https://releases.hashicorp.com/terraform/0.12.4/terraform_0.12.4_linux_amd64.zip
RUN unzip /tmp/terraform.zip -d /usr/bin/
