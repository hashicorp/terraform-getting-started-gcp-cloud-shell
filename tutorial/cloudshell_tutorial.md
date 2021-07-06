# Getting Started with Terraform on Google Cloud Platform

## Introduction

### About this Tutorial

This tutorial will help you learn the fundamentals of Terraform and how
to provision infrastructure on Google Cloud Platform (GCP).

In these tutorials, you will use Terraform to
provision, update, and destroy a simple set of infrastructure using the sample
configuration provided. The sample configuration provisions a network and a
Linux virtual machine. You will also learn about remote backends, input and
output variables, and how to configure resource dependencies. These are the
building blocks for more complex configurations.

**Warning:** While everything provisioned in this tutorial should fall within
  GCP's free tier, if you provision resources outside of the free tier, you may
  be charged. We are not responsible for any charges you may incur.

### Google Cloud Shell

This tutorial uses Google Cloud Shell to give you an environment preconfigured
with Terraform. You can execute commands at the command prompt, and edit the
files in the editor window.

If you prefer to follow this tutorial on your local machine, you can follow
the [Google Cloud collection on
learn.hashicorp.com](https://learn.hashicorp.com/collections/terraform/gcp-get-started).

## Prerequisites

### Terraform

Terraform is already installed in your Cloud Shell environment. You can verify
this with the `terraform version`.

```bash
terraform version
```

**Note:** When you run the previous command, Terraform may print a warning that
  there is a newer version of Terraform available. This tutorial has been tested
  with the version of Terraform installed in your Cloud Shell environment, so
  you can continue to use it for the rest of the tutorial.

### Create a GCP Project

In addition to a GCP account, you will need to use a **GCP Project** to follow
this guide.

In order to keep the resources you will create in this tutorial separate from
your other resources, we recommend that you create a new project to use
throughout the tutorial.

After you have created the project, select it below.

<walkthrough-project-billing-setup></walkthrough-project-billing-setup>

Selecting the project here will set the project ID in future steps, so 
select a project before moving on!

#### Authentication

When you are using Google Cloud Shell, the shell is already configured with
access to your GCP credentials, so you will not need to do anything extra to
authenticate and start provisioning resources. When using Terraform from another
environment, you will need to configure authentication. You can read about
credentials in the [GCP provider
documentation](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/provider_reference#authentication).

## Build Infrastructure

The set of files used to describe infrastructure in Terraform is known as a
Terraform _configuration_. Next, you will write your first configuration to create a network.

Each Terraform configuration must be in its own working directory. Your Cloud
Shell environment includes a directory called `tutorial` that will store the
configuration you will use for this tutorial.

Terraform loads all files ending in `.tf` or `.tf.json` in the working
directory.

In the `tutorial` directory you will find a file named <walkthrough-editor-open-file filePath="terraform-getting-started-gcp-cloud-shell/tutorial/main.tf">main.tf</walkthrough-editor-open-file>.

Open `main.tf` in the Cloud Shell editor, and paste in the configuration below.

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "3.73.0"
    }
  }

  required_version = ">= 0.15.0"
}

provider "google" {
  project = "{{project-id}}"
  region  = "us-central1"
  zone    = "us-central1-c"
}

module "project_services" {
  source  = "terraform-google-modules/project-factory/google//modules/project_services"
  version = "3.3.0"

  project_id = "{{project-id}}"

  activate_apis = [
    "compute.googleapis.com",
    "oslogin.googleapis.com"
  ]

  disable_services_on_destroy = false
  disable_dependent_services  = false
}
```

This is a complete configuration that Terraform can apply. In the following
sections you will review each block of the configuration in more detail.

Cloud Shell automatically sets the `project_id` attribute to the one you selected
in the previous step. You can review a list of your projects in the [cloud
resource manager](https://console.cloud.google.com/cloud-resource-manager)

### Terraform Block

The `terraform {}` block contains Terraform settings, including the required
providers Terraform will use to provision your infrastructure. For each
provider, the `source` attribute defines an optional hostname, a namespace, and
the provider type. Terraform installs providers from the [Terraform
Registry](https://registry.terraform.io/) by default. In this example
configuration, the `google` provider's source is defined as `hashicorp/google`,
which is shorthand for `registry.terraform.io/hashicorp/google`.

You can also define a version constraint for each provider in the
`required_providers` block. The `version` attribute is optional, but we
recommend using it to enforce the provider version. Without it, Terraform will
always use the latest version of the provider, which may introduce breaking
changes.

To learn more, reference the [provider source
documentation](https://www.terraform.io/docs/language/providers/requirements.html).

### Providers

The `provider` block configures the specified provider, in this case `google`. A
provider is a plugin that Terraform uses to create and manage your resources.
You can define multiple provider blocks in a Terraform configuration to manage
resources from different providers.

### Project Services Module

In this tutorial, you will use the Google Compute Engine service to
provision your network and instance. You also need to enable the OS Login
service for authentication. You must enable these services for your project
before you can use them. This example configuration uses a module to manage your
project services. You can learn more about how this module works from the
[module
documentation](https://registry.terraform.io/modules/terraform-google-modules/project-factory/google/3.3.0/submodules/project_services).

The `project_services` module will enable the listed services for your project.

### Initialize the directory

When you create a new configuration — or check out an existing configuration
from version control — you need to initialize the directory with `terraform
init`. This step downloads the providers defined in the configuration.

Initialize the directory.

```bash
terraform init
```

Terraform returns output similar to the following.

```raw
Initializing modules...
Downloading terraform-google-modules/project-factory/google 3.3.0 for project_services...
- project_services in .terraform/modules/project_services/modules/project_services

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/google versions matching "3.73.0"...
- Installing hashicorp/google v3.73.0...
- Installed hashicorp/google v3.73.0 (self-signed, key ID 34365D9472D7468F)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Terraform downloads the `google` provider and installs it in a hidden
subdirectory of your current working directory, named `.terraform`. The
`terraform init` command prints the provider version Terraform installed.
Terraform also creates a lock file named `.terraform.lock.hcl`, which specifies
the exact provider versions used to ensure that every Terraform run is
consistent. This also allows you to control when you want to upgrade the
providers used in your configuration.

### Format and validate the configuration

We recommend using consistent formatting in all of your configuration files. The
`terraform fmt` command automatically updates configurations in the current
directory for readability and consistency.

Format your configuration. Terraform will print out the names of the files it
modified, if any. In this case, your configuration file was already formatted
correctly, so Terraform will not return any file names.

```bash
terraform fmt
```

You can also make sure your configuration is syntactically valid and internally
consistent by using the `terraform validate` command.

Validate your configuration. The example configuration provided above is valid,
so Terraform will return a success message.

```bash
terraform validate
```

Terraform will confirm that the configuration is valid. 

```raw
Success! The configuration is valid.
```

### Apply Configuration

Apply the configuration now with the `terraform apply` command. Terraform will
print output similar to what is shown below.

**Note:** Google Cloud Shell may prompt you to authorize this action before you
  can continue with the tutorial. Do so now.

```bash
terraform apply
```

Terraform returns output similar to to the output below.

```raw
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # module.project_services.google_project_service.project_services[0] will be created
  + resource "google_project_service" "project_services" {
      + disable_dependent_services = false
      + disable_on_destroy         = false
      + id                         = (known after apply)
      + project                    = "rln-gcp-test-01"
      + service                    = "compute.googleapis.com"
    }

  # module.project_services.google_project_service.project_services[1] will be created
  + resource "google_project_service" "project_services" {
      + disable_dependent_services = false
      + disable_on_destroy         = false
      + id                         = (known after apply)
      + project                    = "rln-gcp-test-01"
      + service                    = "oslogin.googleapis.com"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Respond with a `yes` to have Terraform enable these services.

```raw
  Enter a value: yes

module.project_services.google_project_service.project_services[0]: Creating...
module.project_services.google_project_service.project_services[1]: Creating...
module.project_services.google_project_service.project_services[1]: Creation complete after 3s [id=rln-gcp-test-01/oslogin.googleapis.com]
module.project_services.google_project_service.project_services[0]: Creation complete after 3s [id=rln-gcp-test-01/compute.googleapis.com]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

## Create infrastructure

Now create your first infrastructure, a Virtual Private Cloud (VPC) network that
will contain the rest of the infrastructure for this tutorial. Add the following
to `main.tf`.

```hcl
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
```

Use `resource` blocks to define components of your infrastructure. A
resource might be a physical component such as a server, or it can be a logical
resource such as a Heroku application.

Resource blocks have two strings before the block: the resource type and the
resource name. In this example, the resource type is `google_compute_network` and the name is `vpc_network`. The prefix of the type maps to the name of the provider. In the
example configuration, Terraform manages the `google_compute_network` resource with the
`google` provider. Together, the resource type and resource name form a unique ID
for the resource. For example, the ID for your network is
`google_compute_network.vpc_network`.

Resource blocks contain arguments to configure the resource.
Arguments can include things like machine sizes, disk image names, or VPC IDs.
The [Terraform Registry GCP documentation page](https://registry.terraform.io/providers/hashicorp/google/latest/docs) documents the required and optional arguments for each GCP resource. For example, you can read the [`google_compute_network`](https://www.terraform.io/docs/providers/google/r/compute_network.html) documentation to view the resource's supported arguments and available attributes.

The [GCP provider](https://www.terraform.io/docs/providers/google/index.html)
documents supported resources, including
[google_compute_network](https://www.terraform.io/docs/providers/google/r/compute_network.html) and its supported arguments.

### Apply Configuration

Apply the configuration now with the `terraform apply` command. Terraform will
print output similar to what is shown below. We have truncated some of the
output for brevity.

```bash
terraform apply
```

Terraform will print output similar to what is shown below. We have truncated
some of the output for brevity.

```raw
module.project_services.google_project_service.project_services[1]: Refreshing state... [id=rln-gcp-test-01/oslogin.googleapis.com]
module.project_services.google_project_service.project_services[0]: Refreshing state... [id=rln-gcp-test-01/compute.googleapis.com]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_network.vpc_network will be created
  + resource "google_compute_network" "vpc_network" {
      + auto_create_subnetworks         = true
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "terraform-network"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Terraform returns the planned infrastructure changes, and prompts
for your approval before it makes those changes.

This output shows the _execution plan_, describing which actions Terraform will
take in order to create infrastructure to match the configuration. The output
format is similar to the diff format generated by tools such as Git. The output
has a `+` next to `resource "google_compute_network" "vpc_network"`, meaning
that Terraform will create this resource. Beneath that, it shows the attributes
that will be set. When the value displayed is `(known after apply)`, it means
that the value will not be known until the resource is created.

Terraform will now pause and wait for approval before proceeding. If anything in
the plan seems incorrect or dangerous, it is safe to abort here with no changes
made to your infrastructure.

In this case the plan looks acceptable, so type `yes` at the confirmation prompt
to proceed. It may take a few minutes for Terraform to provision the network.

```raw
  Enter a value: yes

google_compute_network.vpc_network: Creating...
google_compute_network.vpc_network: Still creating... [10s elapsed]
google_compute_network.vpc_network: Still creating... [20s elapsed]
google_compute_network.vpc_network: Still creating... [30s elapsed]
google_compute_network.vpc_network: Still creating... [40s elapsed]
google_compute_network.vpc_network: Creation complete after 42s [id=projects/rln-gcp-test-01/global/networks/terraform-network]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

You created GCP infrastructure using Terraform! Visit the GCP console to
see the network you provisioned. Make sure you are looking at the same
region and project that you configured in the provider configuration.

### Inspect State

When you applied your configuration, Terraform wrote data into a file called
`terraform.tfstate`. Terraform stores the IDs and properties of the resources it
manages in this file, so that it can update or destroy those resources going
forward.

Terraform tracks resources with the Terraform state file. The state file often contains sensitive information, so you must store your state
file securely and distribute it only to trusted team members who need to manage
your infrastructure. In production, we recommend [storing your state
remotely](https://learn.hashicorp.com/tutorials/terraform/cloud-migrate?in=terraform/cloud) with Terraform
Cloud or Terraform Enterprise. Terraform also supports several other [remote
backends](https://www.terraform.io/docs/language/settings/backends/index.html)
you can use to store and manage your state.

Inspect the current state using `terraform show`.

```bash
terraform show
```

Terraform will print output similar to the following.

```raw
# module.project_services.google_project_service.project_services[0]:
resource "google_project_service" "project_services" {
    disable_dependent_services = false
    disable_on_destroy         = false
    id                         = "rln-gcp-test-01/compute.googleapis.com"
    project                    = "rln-gcp-test-01"
    service                    = "compute.googleapis.com"
}

# module.project_services.google_project_service.project_services[1]:
resource "google_project_service" "project_services" {
    disable_dependent_services = false
    disable_on_destroy         = false
    id                         = "rln-gcp-test-01/oslogin.googleapis.com"
    project                    = "rln-gcp-test-01"
    service                    = "oslogin.googleapis.com"
}


# google_compute_network.vpc_network:
resource "google_compute_network" "vpc_network" {
    auto_create_subnetworks         = true
    delete_default_routes_on_create = false
    id                              = "projects/rln-gcp-test-01/global/networks/terraform-network"
    mtu                             = 0
    name                            = "terraform-network"
    project                         = "rln-gcp-test-01"
    routing_mode                    = "REGIONAL"
    self_link                       = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/global/networks/terraform-network"
}
```

When Terraform created these resources, it also gathered its metadata from the
Google provider and recorded it in the state file. Later in this collection, you
will modify your configuration to reference these values to configure other
resources or outputs.

## Change Infrastructure

In the previous section, you created your first infrastructure with Terraform: a
VPC network. In this section, you will modify your configuration, and learn how
to apply changes to your Terraform projects.

Infrastructure is continuously evolving, and Terraform was built to help manage
resources over their lifecycle. When you update Terraform configurations,
Terraform builds an execution plan that only modifies what is necessary to reach
your desired state.

When using Terraform in production, we recommend that you use a version control
system to manage your configuration files, and store your state in a remote
backend such as Terraform Cloud or Terraform Enterprise.

### Create a new resource

You can create new resources by adding them to your Terraform configuration and
running `terraform apply` to provision them.

Add the following configuration for a Google compute instance resource to
`main.tf`.

```hcl
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }

  allow_stopping_for_update = true
}
```

**Note:** The order in which resources are defined in your configuration files
  does not affect how Terraform provisions your resources, so you should
  organize your configuration however is easiest for you and your team to
  manage.

This new resource includes a few arguments. The name and machine type are
simple strings, but `boot_disk` and `network_interface` are more complex blocks.
You can see all of the supported arguments for the resource [in the
GCP provider documentation](https://www.terraform.io/docs/providers/google/r/compute_instance.html).

Your compute instance will use a Debian operating system, and
will be connected to the VPC Network you created earlier. Notice how this
configuration refers to the network's name property with
`google_compute_network.vpc_network.name`. This establishes a dependency
between the two resources. Terraform builds a dependency graph of the
resources it manages to ensure they are created in appropriate order.

The presence of the `access_config` block, even without any arguments, gives
the VM an external IP address, making it accessible over the internet.

**Tip:** All traffic to instances, even from other instances, is blocked by the
  firewall unless firewall rules are created to allow it. Add a
  [`google_compute_firewall`](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_firewall)
  resource to allow traffic to access your instance.

Now run `terraform apply` to create the compute instance.

```bash
terraform apply
```

Terraform will prompt you to confirm the operation.

```raw
google_compute_network.vpc_network: Refreshing state... [id=projects/rln-gcp-test-01/global/networks/terraform-network]
module.project_services.google_project_service.project_services[1]: Refreshing state... [id=rln-gcp-test-01/oslogin.googleapis.com]
module.project_services.google_project_service.project_services[0]: Refreshing state... [id=rln-gcp-test-01/compute.googleapis.com]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
      + allow_stopping_for_update = true
      + can_ip_forward            = false
      + cpu_platform              = (known after apply)
      + current_status            = (known after apply)
      + deletion_protection       = false
      + guest_accelerator         = (known after apply)
      + id                        = (known after apply)
      + instance_id               = (known after apply)
      + label_fingerprint         = (known after apply)
      + machine_type              = "f1-micro"
      + metadata_fingerprint      = (known after apply)
      + min_cpu_platform          = (known after apply)
      + name                      = "terraform-instance"
      + project                   = (known after apply)
      + self_link                 = (known after apply)
      + tags_fingerprint          = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "debian-cloud/debian-9"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + name               = (known after apply)
          + network            = "terraform-network"
          + network_ip         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart   = (known after apply)
          + min_node_cpus       = (known after apply)
          + on_host_maintenance = (known after apply)
          + preemptible         = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Once again, answer `yes` to the confirmation prompt.

```raw
  Enter a value: yes

google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Creation complete after 19s [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

With this change you added a `google_compute_instance`
resource named `vm_instance` to the configuration, and Terraform created the
resource in GCP.

### Modify configuration

In addition to creating resources, Terraform can also make changes to those
resources.

Add a `tags` argument to your `vm_instance` resource block.

**Tip:** The below snippet is formatted as a diff to give you context about what
in your configuration should change. Add the tags attribute as shown below
(exclude the leading `+` sign).

```diff
 resource "google_compute_instance" "vm_instance" {
   name         = "terraform-instance"
   machine_type = "f1-micro"
+  tags         = ["web", "dev"]
# ...
 }
```

Run `terraform apply` again.

```bash
terraform apply
```

Terraform will print output similar to the following.

```raw
module.project_services.google_project_service.project_services[1]: Refreshing state... [id=rln-gcp-test-01/oslogin.googleapis.com]
google_compute_network.vpc_network: Refreshing state... [id=projects/rln-gcp-test-01/global/networks/terraform-network]
module.project_services.google_project_service.project_services[0]: Refreshing state... [id=rln-gcp-test-01/compute.googleapis.com]
google_compute_instance.vm_instance: Refreshing state... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be updated in-place
  ~ resource "google_compute_instance" "vm_instance" {
        id                        = "projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance"
        name                      = "terraform-instance"
      ~ tags                      = [
          + "dev",
          + "web",
        ]
        # (18 unchanged attributes hidden)



        # (3 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

The prefix `~` means that Terraform will update the resource in-place. You can
apply this change now by responding `yes`, and Terraform will add the tags to
your instance.

### Introduce destructive changes

A destructive change is a change that requires the provider to replace the
existing resource rather than updating it. This usually happens because the
cloud provider does not support updating the resource in the way described by
your configuration.

Changing the disk image of your instance is one example of a destructive change.
Edit the `boot_disk` block inside the `vm_instance` resource in your
configuration file to change the image parameter as follows.

**Tip:** The below snippet is formatted as a diff to give you context about what
  in your configuration should change. Replace the old value (indicated with a `-`) with
  the new value (indicated with a `+`).

```diff hideClipboard
   boot_disk {
     initialize_params {
-      image = "debian-cloud/debian-9"
+      image = "cos-cloud/cos-stable"
     }
   }
```

This modification changes the boot disk from a Debian 9 image to Google's
Container-Optimized OS.

Now run `terraform apply` again. Terraform will replace the instance because
Google Cloud Platform does not support replacing the boot disk image on a
running instance.

```bash
terraform apply
```

Terraform will print output similar to the following:

```raw
module.project_services.google_project_service.project_services[1]: Refreshing state... [id=rln-gcp-test-01/oslogin.googleapis.com]
module.project_services.google_project_service.project_services[0]: Refreshing state... [id=rln-gcp-test-01/compute.googleapis.com]
google_compute_network.vpc_network: Refreshing state... [id=projects/rln-gcp-test-01/global/networks/terraform-network]
google_compute_instance.vm_instance: Refreshing state... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # google_compute_instance.vm_instance must be replaced
-/+ resource "google_compute_instance" "vm_instance" {
      ~ cpu_platform              = "Intel Haswell" -> (known after apply)
      ~ current_status            = "RUNNING" -> (known after apply)
      - enable_display            = false -> null
      ~ guest_accelerator         = [] -> (known after apply)
      ~ id                        = "projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
      ~ instance_id               = "4962028063030753807" -> (known after apply)
      ~ label_fingerprint         = "42WmSpB8rSM=" -> (known after apply)
      - labels                    = {} -> null
      - metadata                  = {} -> null
      ~ metadata_fingerprint      = "qO3Rix9hoJg=" -> (known after apply)
      + min_cpu_platform          = (known after apply)
        name                      = "terraform-instance"
      ~ project                   = "rln-gcp-test-01" -> (known after apply)
      - resource_policies         = [] -> null
      ~ self_link                 = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance" -> (known after apply)
        tags                      = [
            "dev",
            "web",
        ]
      ~ tags_fingerprint          = "XaeQnaHMn9Y=" -> (known after apply)
      ~ zone                      = "us-central1-c" -> (known after apply)
        # (4 unchanged attributes hidden)

      ~ boot_disk {
          ~ device_name                = "persistent-disk-0" -> (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          ~ source                     = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/zones/us-central1-c/disks/terraform-instance" -> (known after apply)
            # (2 unchanged attributes hidden)

          ~ initialize_params {
              ~ image  = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-9-stretch-v20210609" -> "cos-cloud/cos-stable" # forces replacement
              ~ labels = {} -> (known after apply)
              ~ size   = 10 -> (known after apply)
              ~ type   = "pd-standard" -> (known after apply)
            }
        }

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      ~ network_interface {
          ~ name               = "nic0" -> (known after apply)
          ~ network            = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/global/networks/terraform-network" -> "terraform-network"
          ~ network_ip         = "10.128.0.2" -> (known after apply)
          ~ subnetwork         = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/regions/us-central1/subnetworks/terraform-network" -> (known after apply)
          ~ subnetwork_project = "rln-gcp-test-01" -> (known after apply)

          ~ access_config {
              ~ nat_ip       = "35.222.158.251" -> (known after apply)
              ~ network_tier = "PREMIUM" -> (known after apply)
            }
        }

      ~ scheduling {
          ~ automatic_restart   = true -> (known after apply)
          ~ min_node_cpus       = 0 -> (known after apply)
          ~ on_host_maintenance = "MIGRATE" -> (known after apply)
          ~ preemptible         = false -> (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

The prefix `-/+` means that Terraform will destroy and recreate the resource,
rather than updating it in-place. Terraform and the GCP provider handle these
details for you, and the execution plan reports what Terraform will do.

Additionally, the execution plan shows that the disk image change is the modification that
forced the instance replacement. Using this information, you can adjust
your changes to possibly avoid destructive updates if they are not acceptable.

Once again, Terraform prompts for approval of the execution plan before
proceeding. Answer `yes` to execute the planned steps:

```raw
  Enter a value: yes

google_compute_instance.vm_instance: Destroying... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance]
google_compute_instance.vm_instance: Still destroying... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Still destroying... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance, 20s elapsed]
google_compute_instance.vm_instance: Destruction complete after 21s
google_compute_instance.vm_instance: Creating...
google_compute_instance.vm_instance: Still creating... [10s elapsed]
google_compute_instance.vm_instance: Creation complete after 13s [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
```

As indicated by the execution plan, Terraform first destroyed the existing
instance and then created a new one in its place. You can use `terraform show`
again to see the new values associated with this instance.

## Destroy Infrastructure

You have created and modified infrastructure using Terraform. You will now destroy your Terraform-managed infrastructure.

Once you no longer need infrastructure, you might want to destroy it to reduce
your security exposure and costs. For example, you may remove a production
environment from service, or manage short-lived environments like build or
testing systems. In addition to building and modifying infrastructure, Terraform
can destroy or recreate the resources it manages.

### Destroy

The `terraform destroy` command terminates resources managed by your Terraform
project. This command terminates
all the resources specified in your Terraform state. It does _not_ destroy
resources running elsewhere that are not managed by the current Terraform
project.

```bash
terraform destroy
```

As with apply, Terraform shows its execution plan and waits for approval before
making any changes.

```raw
google_compute_network.vpc_network: Refreshing state... [id=projects/rln-gcp-test-01/global/networks/terraform-network]
module.project_services.google_project_service.project_services[0]: Refreshing state... [id=rln-gcp-test-01/compute.googleapis.com]
module.project_services.google_project_service.project_services[1]: Refreshing state... [id=rln-gcp-test-01/oslogin.googleapis.com]
google_compute_instance.vm_instance: Refreshing state... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  - destroy

Terraform will perform the following actions:

  # google_compute_instance.vm_instance will be destroyed
  - resource "google_compute_instance" "vm_instance" {
      - allow_stopping_for_update = true -> null
      - can_ip_forward            = false -> null
      - cpu_platform              = "Intel Haswell" -> null
      - current_status            = "RUNNING" -> null
      - deletion_protection       = false -> null
      - enable_display            = false -> null
      - guest_accelerator         = [] -> null
      - id                        = "projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance" -> null
      - instance_id               = "6044171508781660376" -> null
      - label_fingerprint         = "42WmSpB8rSM=" -> null
      - labels                    = {} -> null
      - machine_type              = "f1-micro" -> null
      - metadata                  = {} -> null
      - metadata_fingerprint      = "qO3Rix9hoJg=" -> null
      - name                      = "terraform-instance" -> null
      - project                   = "rln-gcp-test-01" -> null
      - resource_policies         = [] -> null
      - self_link                 = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance" -> null
      - tags                      = [
          - "dev",
          - "web",
        ] -> null
      - tags_fingerprint          = "XaeQnaHMn9Y=" -> null
      - zone                      = "us-central1-c" -> null

      - boot_disk {
          - auto_delete = true -> null
          - device_name = "persistent-disk-0" -> null
          - mode        = "READ_WRITE" -> null
          - source      = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/zones/us-central1-c/disks/terraform-instance" -> null

          - initialize_params {
              - image  = "https://www.googleapis.com/compute/v1/projects/cos-cloud/global/images/cos-stable-89-16108-470-1" -> null
              - labels = {} -> null
              - size   = 10 -> null
              - type   = "pd-standard" -> null
            }
        }

      - network_interface {
          - name               = "nic0" -> null
          - network            = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/global/networks/terraform-network" -> null
          - network_ip         = "10.128.0.3" -> null
          - subnetwork         = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/regions/us-central1/subnetworks/terraform-network" -> null
          - subnetwork_project = "rln-gcp-test-01" -> null

          - access_config {
              - nat_ip       = "35.232.29.223" -> null
              - network_tier = "PREMIUM" -> null
            }
        }

      - scheduling {
          - automatic_restart   = true -> null
          - min_node_cpus       = 0 -> null
          - on_host_maintenance = "MIGRATE" -> null
          - preemptible         = false -> null
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

  # google_compute_network.vpc_network will be destroyed
  - resource "google_compute_network" "vpc_network" {
      - auto_create_subnetworks         = true -> null
      - delete_default_routes_on_create = false -> null
      - id                              = "projects/rln-gcp-test-01/global/networks/terraform-network" -> null
      - mtu                             = 0 -> null
      - name                            = "terraform-network" -> null
      - project                         = "rln-gcp-test-01" -> null
      - routing_mode                    = "REGIONAL" -> null
      - self_link                       = "https://www.googleapis.com/compute/v1/projects/rln-gcp-test-01/global/networks/terraform-network" -> null
    }

  # module.project_services.google_project_service.project_services[0] will be destroyed
  - resource "google_project_service" "project_services" {
      - disable_dependent_services = false -> null
      - disable_on_destroy         = false -> null
      - id                         = "rln-gcp-test-01/compute.googleapis.com" -> null
      - project                    = "rln-gcp-test-01" -> null
      - service                    = "compute.googleapis.com" -> null
    }

  # module.project_services.google_project_service.project_services[1] will be destroyed
  - resource "google_project_service" "project_services" {
      - disable_dependent_services = false -> null
      - disable_on_destroy         = false -> null
      - id                         = "rln-gcp-test-01/oslogin.googleapis.com" -> null
      - project                    = "rln-gcp-test-01" -> null
      - service                    = "oslogin.googleapis.com" -> null
    }

Plan: 0 to add, 0 to change, 4 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

The `-` prefix indicates that Terraform will destroy these resources.

Answer `yes` to execute this plan and destroy the infrastructure.

```raw
  Enter a value: yes

module.project_services.google_project_service.project_services[1]: Destroying... [id=rln-gcp-test-01/oslogin.googleapis.com]
module.project_services.google_project_service.project_services[0]: Destroying... [id=rln-gcp-test-01/compute.googleapis.com]
module.project_services.google_project_service.project_services[1]: Destruction complete after 0s
google_compute_instance.vm_instance: Destroying... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance]
module.project_services.google_project_service.project_services[0]: Destruction complete after 0s
google_compute_instance.vm_instance: Still destroying... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance, 10s elapsed]
google_compute_instance.vm_instance: Still destroying... [id=projects/rln-gcp-test-01/zones/us-central1-c/instances/terraform-instance, 20s elapsed]
google_compute_instance.vm_instance: Destruction complete after 22s
google_compute_network.vpc_network: Destroying... [id=projects/rln-gcp-test-01/global/networks/terraform-network]
google_compute_network.vpc_network: Still destroying... [id=projects/rln-gcp-test-01/global/networks/terraform-network, 10s elapsed]
google_compute_network.vpc_network: Still destroying... [id=projects/rln-gcp-test-01/global/networks/terraform-network, 20s elapsed]
google_compute_network.vpc_network: Still destroying... [id=projects/rln-gcp-test-01/global/networks/terraform-network, 30s elapsed]
google_compute_network.vpc_network: Still destroying... [id=projects/rln-gcp-test-01/global/networks/terraform-network, 40s elapsed]
google_compute_network.vpc_network: Destruction complete after 41s

Destroy complete! Resources: 4 destroyed.
```

As with `terraform apply`, Terraform determines the order in which resources
must be destroyed. GCP will not destroy a VPC network if there are other
resources still in it, so Terraform waits until the instance is destroyed first.
When performing operations, Terraform creates a dependency graph to determine
the correct order of operations. In more complicated cases with multiple
resources, Terraform will perform operations in parallel when it's safe to do
so.

## Define Input Variables

You now have enough Terraform knowledge to create useful configurations, but the
examples so far have used hard-coded values. Terraform configurations can
include variables to make your configuration more dynamic and flexible.

Edit the file called `variables.tf` to add the following variable definitions.

```hcl
variable "project" { }

variable "region" {
  default = "us-central1"
}

variable "zone" {
  default = "us-central1-c"
}
```

**Tip:** Terraform loads all files ending in `.tf` in the working directory, so
  you can name your configuration files however you choose. We recommend
  defining variables in their own file to make your configuration easier to
  organize and understand.

This file defines four variables within your Terraform configuration. The
`project` and `credentials_file` variables have an empty block: `{ }`. The
`region` and `zone` variables set defaults. If a default value is set, the
variable is optional. Otherwise, the variable is required. If you run `terraform
plan` now, Terraform will prompt you for the values for `project` and
`credentials_file`.

### Use variables in configuration

Next, update the GCP provider configuration in `main.tf` to use these new
variables.

Replace the entire `provider "google"` block with the following.

```hcl
provider "google" {
  project = var.project
  region  = var.region
  zone    = var.zone
}
```

Variables are referenced with the `var.` prefix.

### Assign values to your variables

You can populate variables using values from a file. Terraform automatically
loads files called `terraform.tfvars` or matching `*.auto.tfvars` in the working
directory when running operations.

Edit the file called `terraform.tfvars` and copy and paste the value below.

```hcl
project = "{{project-id}}"
```

Save this file.

### Apply configuration

Now run `terraform apply`.

```bash
terraform apply
```

As before, respond to the confirmation prompt with a `yes`.

Since the region and zone input variables are configured with defaults, you do
not need to set them. The values you set in your variables file match the
original configuration, so Terraform does not need to make any changes to your
infrastructure.

Terraform supports many ways to use and set variables. To learn more, follow our
in-depth tutorial, [Customize Terraform Configuration with
Variables](https://learn.hashicorp.com/tutorials/terraform/variables?in=terraform/configuration-language).

## Query Data with Output Variables

In the previous section, you used input variables to parameterize your Terraform
configuration. In this section, you will use output values to organize data to
be easily queried and displayed to the Terraform user.

When building complex infrastructure, Terraform stores hundreds or thousands of
attribute values for all your resources. As a user of Terraform, you may only
be interested in a few values of importance. Outputs designate which data to
display. This data is outputted when `apply` is called, and can be queried
using the `terraform output` command.

### Define Outputs

Define an output for the IP address of the instance that Terraform provisions. Create
a file called `outputs.tf` with the following contents.

```hcl
output "ip" {
  value = google_compute_instance.vm_instance.network_interface.0.network_ip
}
```

### Inspect outputs

You must apply this configuration before you can use these output values. Apply
your configuration now.

```bash
terraform apply
```

Respond to the confirmation prompt with `yes`.

```raw
Plan: 0 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + ip = "10.128.0.2"

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes


Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

ip = "10.128.0.2"
```

Now query the outputs with the `terraform output` command.

```bash
terraform output
```

Terraform will print output similar to the following.

```raw
ip = "10.128.0.2"
```

You can use Terraform outputs to connect your Terraform projects with other
parts of your infrastructure, or with other Terraform projects. To learn more,
follow our in-depth tutorial, [Output Data from Terraform](https://learn.hashicorp.com/tutorials/terraform/outputs?in=terraform/configuration-language).

## Destroy your infrastructure

Make sure to run `terraform destroy` to clean up the resources you created in
these tutorials.

```bash
terraform destroy
```

When prompted remember to confirm with a `yes`.

## Next steps

That concludes the getting started tutorial for Terraform. Hopefully you're now
able to not only see what Terraform is useful for, but you're also able to put
this knowledge to use to improve building your own infrastructure.

For more hands-on experience with the Terraform configuration language, or to
learn more of the building blocks of Terraform, review the tutorials below.

- [Configuration Language](https://learn.hashicorp.com/collections/terraform/configuration-language) - Get more familiar with variables, outputs, dependencies, meta-arguments, and other language features to write more sophisticated Terraform configurations.

- [Modules](https://learn.hashicorp.com/tutorials/terraform/module) - Organize and re-use Terraform configuration with modules.

- [Provision](https://learn.hashicorp.com/collections/terraform/provision) - Use Packer or Cloud-init to automatically provision SSH keys and a web server onto a Linux VM created by Terraform in AWS.

- [Import](https://learn.hashicorp.com/tutorials/terraform/state-import) - Import existing infrastructure into Terraform.

To read more about available configuration options, explore the [Terraform documentation](https://www.terraform.io/docs/index.html).

### Learn more about Terraform Cloud

Although Terraform Cloud can act as a standard remote backend to support
Terraform runs on local machines, it works even better as a remote run
environment. It supports two main workflows for performing Terraform runs:

- A VCS-driven workflow, in which it automatically queues plans whenever changes
  are committed to your configuration's VCS repo.
- An API-driven workflow, in which a CI pipeline or other automated tool can
  upload configurations directly.

For a hands-on introduction to the Terraform Cloud VCS-driven workflow, [follow the Terraform Cloud
getting started tutorials](https://learn.hashicorp.com/collections/terraform/cloud-get-started). Terraform Cloud also offers [commercial solutions](https://www.hashicorp.com/products/terraform/pricing) which include team permission management, policy enforcement, agents, and more.
