# Getting Started with Terraform

Terraform is a tool that allows you to define and manage infrastructure using a configuration driven workflow known as infrastructure as code (IaC).

## Learning Objectives

By the end of this guide, you will be able to:

- Install Terraform on your local machine
- Create a Terraform configuration file with a provider and resources
- Initialize a Terraform project and install required providers
- Provision resources using the `terraform apply` command
- Remove resources using the `terraform destroy` command
- Understand the basics of how Terraform manages the resource lifecycle

## Prerequisites

Before you start, you need:

- A Mac or Linux system (Windows users can use [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install))
- Docker installed and running on your system (see [Docker installation guide](https://docs.docker.com/get-docker/))
  - Your user must be able to run Docker commands (in the docker group)
- A text editor (VS Code, Vim, Nano, etc.)
- Basic familiarity with the command line interface (CLI)

## Install Terraform

You can install Terraform using two methods:

1. **Package manager (recommended)** - Your system's package manager handles binary PATH configuration automatically. This is the best option for most users.
2. **Manual binary download** - Download the binary directly and manage PATH configuration yourself.

To install Terraform, visit the [official Terraform installation guide](https://developer.hashicorp.com/terraform/install) and follow the instructions for your preferred method and platform.

After installation, validate that Terraform installed correctly by running the following command in your terminal:

```shell
$ terraform -v
```

You should see output similar to:

```
Terraform v1.9.0
on darwin_arm64
```

## Create Your First Terraform Configuration

In this section, you create a Terraform configuration that provisions an Nginx web server running in a Docker container.

### Understand Terraform Concepts

Before you write configuration code, understand these key Terraform concepts:

- **Provider** - A plugin that Terraform uses to interact with a platform (like Docker, AWS, or Kubernetes). In this guide, you use the Docker provider.
- **Resource** - An infrastructure object managed by Terraform (like a Docker container or image). You define resources in your configuration.
- **Configuration file** - A file with the `.tf` extension that contains your infrastructure definitions written in HCL (HashiCorp Configuration Language).

### Set Up Your Project Directory

Create a new directory on your local machine for this project:

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

Next, create a file named `main.tf` for your Terraform configuration:

```shell
$ touch main.tf
```

### Add Configuration Code

Open `main.tf` in your text editor and paste the following configuration:

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

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "training"
  ports {
    internal = 80
    external = 8080
  }
}
```

> **Note on Docker socket:** This guide uses the default Docker socket. If you use Rancher Desktop, Colima, or Podman, find your socket path with `docker context inspect` and look for the `Host` field. Then update the `host` line:
> ```
> host = "unix:///path/to/your/docker.sock"
> ```

This configuration defines:

- **terraform block** - Declares that your configuration requires the Docker provider
- **provider "docker" block** - Tells Terraform how to connect to your Docker daemon
- **docker_image resource** - Pulls the Nginx image from Docker Hub
- **docker_container resource** - Starts a Docker container running Nginx and maps port 8080

## Initialize Terraform

Initialize your Terraform project with the `init` command. This command downloads the provider plugin and prepares your project:

```shell
$ terraform init
```

You should see output similar to:

```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (unauthenticated)

Terraform has been successfully initialized!
```

The `init` command should complete successfully. If it fails, see the [Terraform troubleshooting documentation](https://developer.hashicorp.com/terraform/docs/language/settings/backends/configuration) for help resolving provider or backend configuration issues.

## Provision the Resources

Now provision the resources by running the `apply` command:

```shell
$ terraform apply
```

Terraform displays the changes it will make. Review the output and type `yes` when prompted to confirm:

```
docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 15s [id=sha256:abc123...]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 2s [id=abc123def456...]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Terraform has created the Docker image and started the container. You can now access Nginx by opening `http://localhost:8080` in your web browser.

> **Troubleshooting:** If the `apply` command fails with an error, review the error message and check that your configuration matches the example above. If your syntax is correct but the command still fails, see the [Terraform troubleshooting documentation](https://developer.hashicorp.com/terraform/docs/troubleshoot) for help with provider or resource-specific issues.

## Remove the Resources

After you verify the container is running and accessible, clean up the resources you created using the `destroy` command:

```shell
$ terraform destroy
```

At the bottom of the output, Terraform asks for confirmation. Review the resources that will be destroyed and type `yes` to confirm:

```
docker_image.nginx: Destroying... [id=sha256:abc123...]
docker_image.nginx: Destruction complete after 1s
docker_container.nginx: Destroying... [id=abc123def456...]
docker_container.nginx: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
```

Terraform removes the Docker container and image from your system.

## Next Steps

You have completed all the learning objectives for this guide: installed Terraform, created a configuration file, initialized your project, and provisioned and removed resources.

For more learning resources, visit:

- [Terraform Documentation](https://developer.hashicorp.com/terraform/docs) - Official Terraform documentation
- [Terraform Learning Path](https://developer.hashicorp.com/terraform/tutorials) - Guided tutorials from HashiCorp
- [Docker Provider Documentation](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs) - Docker provider reference
