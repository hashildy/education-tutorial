# Getting Started with Terraform

## Prerequisites

- Install [Docker Engine or Docker Desktop](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/docker-get-started/#quick-start-tutorial)
- Understanding of computer infrastructure (for example: VMs, containers)
- Comfortable working with CLI (command line interface) commands
- Helpful to have an understanding of DevOps concepts (for example: immutable, infrastructure as code)
  - See the [Next Steps](#next-steps) section for additional resources

## Learning Objectives

In this module, you will: 
  - Install Terraform
  - Create an NGINX (web server) Docker container with Terraform code
  - Delete the Docker container with Terraform code

Infrastructure as code (IaC) is a method of defining and deploying computer infrastructure with easy to understand language. While multiple IaC tools exist, Terraform is among the most popular. This is because Terraform can be used across many different platforms from cloud providers to on-premises solutions. To learn more about Terraform, you can visit one of our [Getting Started Tutorials](https://learn.hashicorp.com/tutorials/terraform/infrastructure-as-code?in=terraform/aws-get-started). 

## Install Terraform

To install Terraform, visit [Terraform.io](https://www.terraform.io/downloads.html) and follow the instructions for your operating system. If you aren't using a package manager, you may have to manually install the binary. You can refer to the [Install Terraform Tutorial](https://learn.hashicorp.com/tutorials/terraform/install-cli) for additional assistance. 

## Create Docker Container with Terraform

With Terraform installed, we are ready to start creating some infrastructure. We recommend creating a new directory on your local machine to store the Terraform configuration code.

```console
$ mkdir terraform-demo
$ cd terraform-demo

~/terraform-demo
```

Next, create a file for your Terraform configuration code.

```console
$ touch main.tf
```

Open main.tf using the text editor of your choice (for example: vi, nano) and paste the following lines into the file. The provided code will define both a provider and a resource. The resource will be provisioned as a Docker container with the latest NGINX web server software. 

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

We are now ready to initialize Terraform with the `terraform init` command. The required Docker provider or plug-in will be installed. 

```console
$ terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v2.16.0...
- Installed kreuzwerker/docker v2.16.0 (self-signed, key ID BD080C4571C6104C)

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

You should check for any errors. For additional assistance troubleshooting errors, you can refer to this [Troubleshoot Terraform Tutorial](https://learn.hashicorp.com/tutorials/terraform/troubleshooting-workflow). If the command ran successfully, provision the resource with the `terraform apply` command.

```console
$ terraform apply
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = (known after apply)
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
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

Terraform will provide an output of the planned actions that will occur once approved. Type `yes` and hit ENTER. The command will take up to a few minutes to run and will display a message indicating that the resource was created.

```console
docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 11s [id=sha256:0e901e68141fd02f237cf63eb842529f8a9500636a9419e3cf4fb986b8fe3d5dnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 2s [id=2ffc17dfa2db904b16c3b48a3f2ad00812e64fbbc7d5fbf28411aeca52cdb523]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

## Validate Docker Container Configuration

There are a couple of ways to confirm that the Docker container was created successfully based on the provided Terraform configuration. 

Run the `docker ps` command which will show any running containers. 

```console
$ docker ps

CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                NAMES
3a3e05c98f62   0e901e68141f   "/docker-entrypoint.…"   12 seconds ago   Up 12 seconds   0.0.0.0:80->80/tcp   training
```

Another way to verify that the Docker container was successfully deployed is to access the local web server that was deployed. You can do this by using the `curl` command for the localhost on port 80. If successful, you will receive an HTML response for the NGINX web server. 

```console
$ curl http://localhost

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Destroy Docker Container with Terraform
Finally, you will destroy the infrastructure by running a `terraform destroy` command.

```console
$ terraform destroy
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach            = false -> null
      - command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      - entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env               = [] -> null
      - gateway           = "172.17.0.1" -> null
      - group_add         = [] -> null
      - hostname          = "2ffc17dfa2db" -> null
      - id                = "2ffc17dfa2db904b16c3b48a3f2ad00812e64fbbc7d5fbf28411aeca52cdb523" -> null
      - image             = "sha256:0e901e68141fd02f237cf63eb842529f8a9500636a9419e3cf4fb986b8fe3d5d" -> null
      - init              = false -> null
      - ip_address        = "172.17.0.2" -> null
      - ip_prefix_length  = 16 -> null
      - ipc_mode          = "private" -> null
      - links             = [] -> null
      - log_driver        = "json-file" -> null
      - log_opts          = {} -> null
      - logs              = false -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      - must_run          = true -> null
      - name              = "training" -> null
      - network_data      = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      - read_only         = false -> null
      - remove_volumes    = true -> null
      - restart           = "no" -> null
      - rm                = false -> null
      - security_opts     = [] -> null
      - shm_size          = 64 -> null
      - start             = true -> null
      - stdin_open        = false -> null
      - storage_opts      = {} -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null
      - tty               = false -> null

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:0e901e68141fd02f237cf63eb842529f8a9500636a9419e3cf4fb986b8fe3d5dnginx:latest" -> null
      - latest      = "sha256:0e901e68141fd02f237cf63eb842529f8a9500636a9419e3cf4fb986b8fe3d5d" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981bafbb884165e514" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: 

```

Look for a message at the bottom of the output asking for confirmation. Type `yes` and hit ENTER. Terraform will destroy the resources it had previously created.

```console
docker_container.nginx: Destroying... [id=2ffc17dfa2db904b16c3b48a3f2ad00812e64fbbc7d5fbf28411aeca52cdb523]
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying... [id=sha256:0e901e68141fd02f237cf63eb842529f8a9500636a9419e3cf4fb986b8fe3d5dnginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

## Next Steps

In this tutorial, you installed Terraform, created a Docker container with NGINX installed using Terraform and destroyed the Docker container using Terraform. To further explore ways to use Terraform, we encourage you to visit the following:

- [Terraform Learn Tutorials](https://learn.hashicorp.com/terraform)
- [Terraform Product Documentation](https://www.terraform.io/docs)
- [Terraform Blogs](https://www.hashicorp.com/blog/products/terraform)

If you are interested in learning about additional DevOps concepts, we encourage you to visit the following: 

- [Why we use Terraform and not Chef, Puppet, Ansible, SaltStack, or CloudFormation](https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c)
- [A Tale of Two Terraforms](https://medium.com/rigged-ops/a-tale-of-two-terraforms-a-model-for-managing-immutable-and-mutable-infrastructure-fa0f5422c27b)
- [Why you should build an immutable infrastructure](https://www.cloudbees.com/blog/immutable-infrastructure)
- [Mutable vs Immutable Infrastructure](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure)