# Getting Started with Terraform

Terraform is the most popular langauge for defining and provisioning infrastructure as code (IaC). In this guide, you will learn how to [setup Terraform](#setup-terraform) on your computer, [create a Terraform configuration](#create-a-terraform-configuration), and perform [infrastructure management](#manage-infrastructure-with-terraform) such as deploying and destorying infrastructure using Terraform.

## Prerequisites

- Computer running Windows, MacOS, Linux, FreeBSD, or Solaris
- [Docker installed](https://docs.docker.com/engine/install/) and configured
- Internet access to download Terraform

## Setup Terraform

To install Terraform, open a browswer and go to [Terraform.io](https://www.terraform.io/downloads.html) and download the zip file for the operating system you will use with Terraform. Extract the binary from the zip file to a directory included in your system's `PATH`.

Verify Terraform is setup correctly.

```shell
$ terraform -version
```
You will see the version and operating system information.

```shell
Terraform v1.0.5
on windows_amd64
```

## Create a Terraform Configuration
Create a new directory on your local machine.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

Create a file for your Terraform configuration code.

```shell
$ touch main.tf
```

Paste the following lines into the main.tf file.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```
## Manage Infrastructure with Terraform
Initialize Terraform with the `init` command. The Docker provider will be installed.

```shell
$ terraform init
```
The following output will be displayed.
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v2.15.0...
- Installed kreuzwerker/docker v2.15.0 (self-signed, key ID BD080C4571C6104C)

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

Check for any errors and correct as necessary. After a successful `init`, you can provision the resource with the `apply` command.

```shell
$ terraform apply
```

The command will take up to a few minutes to run and will display a message indicating that the resource is ready to be created.

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      ...
          }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```
Type `yes` at the `Enter a value:` prompt and press the ENTER button on your keyboard. Terraform will provision the container.

```
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 8s [id=sha256:abcefgdontyouloveiacnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=7cde9cefy8v68451c03e8u89g440fc022899d217bd19a8e]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```
Run `docker ps` to verify the container is running.

```
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                NAMES
7cde9cefce20   dd34e67e3371   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   training
```

Finally, destroy the infrastructure.

```shell
$ terraform destroy
```
Review the actions Terraform will perform.

```
docker_image.nginx: Refreshing state... [id=sha256:dd34e67e3371dc2d1328790c3157ee42dfcae74afffd86b297459ed87a98c0fbnginx:latest]
docker_container.nginx: Refreshing state... [id=7cde9cefce208451c03e8cf03e3c58b7d269fc2f17440fc022899d217bd19a8e]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the last "terraform apply":

  # docker_container.nginx has been changed
  ~ resource "docker_container" "nginx" {
      + dns               = []
      + dns_opts          = []
      + dns_search        = []
      + group_add         = []
        id                = "7cde9cefce208451c03e8cf03e3c58b7d269fc2f17440fc022899d217bd19a8e"
      + links             = []
      + log_opts          = {}
        name              = "training"
      + storage_opts      = {}
      + sysctls           = {}
      + tmpfs             = {}
        # (31 unchanged attributes hidden)

        # (1 unchanged block hidden)
    }

Unless you have made equivalent changes to your configuration, or ignored the relevant attributes using ignore_changes,
the following plan may include actions to undo or respond to these changes.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:dd34e67e3371dc2d1328790c3157ee42dfcae74afffd86b297459ed87a98c0fbnginx:latest" -> null
      - latest      = "sha256:dd34e67e3371dc2d1328790c3157ee42dfcae74afffd86b297459ed87a98c0fb" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:4d4d96ac750af48c6a551d757c1cbfc071692309b491b70b2b8976e102dd3fef" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

Type `yes` and press the ENTER button on your keyboard. Terraform will destroy the container it created.

```
docker_container.nginx: Destroying... [id=7cde9cefce208451c03e8cf03e3c58b7d269fc2f17440fc022899d217bd19a8e]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:dd34e67e3371dc2d1328790c3157ee42dfcae74afffd86b297459ed87a98c0fbnginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

## Next steps

In this section you learned how to setup your local machine with Terraform, create a Terraform configuration, deployed and destroyed infrastructure using Terraform. In the next lesson,  you will learn how to [define input variables](https://learn.hashicorp.com/tutorials/terraform/aws-build?in=terraform/aws-get-started) to make your configuration more dynamic and flexible.