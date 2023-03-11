# Docker Setup

- [Docker Setup](#docker-setup)
  - [Install Docker Desktop on Windows](#install-docker-desktop-on-windows)
  - [Quick start to run a simple app demo on your Docker](#quick-start-to-run-a-simple-app-demo-on-your-docker)
  - [Basical command and concepts in Docker](#basical-command-and-concepts-in-docker)

## Install Docker Desktop on Windows

> Differences between Docker Desktop and Docker?
> Bascially, Docker Desktop is a Docker which has graphical user interface (GUI).

- visit [official website](https://docs.docker.com/desktop/install/windows-install/) for docker and download Docker Desktop for Windows.

## Quick start to run a simple app demo on your Docker

``` bash
# get a app demo from github
git clone https://github.com/docker/getting-started.git
cd getting-started
# create a docker image using the Dockerfile in the app demo directory
docker build -t getting-started .
# run the container using getting-started image in detached mode and map port 80 of the host to port 80 in the container
docker run -dp 80:80 getting-started
```

## Basical command and concepts in Docker

Concepts:

- **Docker images**: An image is a read-only template that contains all the necessary files, configuration settings, and dependencies required to create a container. You can think of an image as a snapshot of a Docker container.

- **Docker containers**: A container is a lightweight and executable package that includes everything needed to run an application, such as code, libraries, and system tools. Containers are created from Docker images and can be started, stopped, and deleted as needed.

- **Docker repositories**: A repository is a collection of Docker images, usually grouped by project or application. Repositories can be hosted in public or private registries, such as Docker Hub or a private registry.

Commands:

- **docker build**: This command is used to build a Docker image from a Dockerfile. A Dockerfile is a script that contains a series of instructions that Docker follows to build the image, including copying files, installing dependencies, and setting up the environment.

- **docker run**: This command is used to start a new Docker container from an existing Docker image. You can specify various options, such as the container name, the port mapping, and the environment variables, to customize the container configuration.

- **docker stop**: This command is used to stop a running container. You can specify the container name or ID to identify the container to stop.

- **docker rm**: This command is used to delete a stopped container. You can specify the container name or ID to identify the container to delete.

- **docker push**: This command is used to push a Docker image to a remote repository, such as Docker Hub or a private registry.

- **docker pull**: This command is used to pull a Docker image from a remote repository to your local machine.
