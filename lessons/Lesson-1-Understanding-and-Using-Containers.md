# Lesson 1 - Understanding and Using Containers

- [Lesson 1 - Understanding and Using Containers](#lesson-1---understanding-and-using-containers)
    - [1.1 What is a Container](#11-what-is-a-container)
      - [What is a Container](#what-is-a-container)
      - [Understanding Container Components](#understanding-container-components)
      - [Containers are Linux](#containers-are-linux)
      - [Understanding Container Runtime](#understanding-container-runtime)
      - [Understanding the OCI](#understanding-the-oci)
      - [Understanding Docker](#understanding-docker)
      - [Understanding to Docker](#understanding-to-docker)
    - [1.2 Understanding Registries](#12-understanding-registries)
      - [Understanding Registries](#understanding-registries)
      - [Accessing Registries](#accessing-registries)
    - [1.3 Starting Containers](#13-starting-containers)
      - [Starting Containers](#starting-containers)
      - [Demo: Installing Docker on Ubuntu](#demo-installing-docker-on-ubuntu)
      - [Demo: Running a Container](#demo-running-a-container)
      - [Running Another Container](#running-another-container)
    - [1.4 Managing Containers](#14-managing-containers)
      - [Common Commands Overview](#common-commands-overview)
      - [Inspecting Container Settings](#inspecting-container-settings)
    - [1.5 Managing Container Images](#15-managing-container-images)
      - [Managing Images](#managing-images)
    - [1.6 Understanding Container Logging](#16-understanding-container-logging)
      - [Understanding Container Logging](#understanding-container-logging)
    - [Lesson 1 Lab Using Containers](#lesson-1-lab-using-containers)
    - [Lesson 1 Lab Solution Using Containers](#lesson-1-lab-solution-using-containers)

### 1.1 What is a Container

#### What is a Container

- A container is a self-contained ready-to-run application.
- This is what makes it different from a virtual machine.
- Containers have all on board that is required to start the application.
- To start a container, a container runtime is required.
- The container runtime is running on a host platform and establishes communication between the local host kernel and the container.
- So, all containers, no matter what they do, run on top of the same local host kernel.

#### Understanding Container Components

- Images are read-only environments that contain the runtime environment, which includes the application and all libraries it requires.
- Containers are the isolated runtime environments where the application is running. By using namespaces the containers can be offered as a strictly isolated environment.
- Registries are used to store images. Docker Hub is a common registry, other registries exist (like quay.io) and private registries can be created also.

#### Containers are Linux

- Containers are based on features offered by the Linux operating system.
- Linux Kernel Namespaces provide strict isolatioin between system components at different levels.
  - network
  - file
  - users
  - processes
  - IPCs
- Linux CGroups offer resource allocation and limitation.

#### Understanding Container Runtime

- The container runtime allows for starting and running the container on top of the host OS.
- The container runtime is responsible for all parts of running the container which are not already a part of the running container program itself.
- Different container runtime solutions exist.
  - Docker
  - lxc
  - runc
  - cri-o
  - containerd
- These runtimes are included in the different container solutions.

#### Understanding the OCI

- OCI the Open Containers Initiative - (https://opencontainers.org)
- It standardizes the use of containers.
  - The image-spec defines how to package a container in a "filesystem bundle"
  - The runtime-spec defines how to run theat filesytem in a container.
- OCI standardization ensures compatibility between containers, no matter which environment they originally come from.
- The result is that for instance images made for Docker work without modifications in Red Hat Podman.   

#### Understanding Docker

- Docker is important in the container landscape, but Docker in NOT the only way to run containers.
- When it started in 2013, Docker offered the following:
  - Container image format.
  - Dockerfile, which is a method for building container images.
  - A way to manage container images.
  - A way to run containers.
  - A way to manage container instances.
  - A solution to share container images.
- As Docker is so common, we will use it to demonstrate containers in this lesson.
- **podman** is the main alternative to Docker.

#### Understanding to Docker

- With the launch of RHEL 8, Red Hat started offering **podman** as an afternative to Docker.
  - **podman** runs containers without the need of having a daemon directly on top of the cri-o container runtime.
  - **buildah** is the related service that is used for managing container images.
- Other solutions for running containers also exist.
  - **LXC** is a Linux-native container runtime.
  - **systemd-nspawn** offers containers integrated in Systemd.

### 1.2 Understanding Registries

#### Understanding Registries

- Container registries are used to provide access to container images.
- Registries make distribution of containers easier: anyone can publish images on public registries.
- Docker Hub (hub.docker.com) is the biggest registry.
- Other registries exist as well, like quay.io.
- It's also possible to create private registries.

#### Accessing Registries

- Access to registries is normally free.
- In some cases, additional authrization is required and you will need an account to access registry contents.
- On Docker Hub, an account gives access to additional features.
  - Higher image pull limit.
  - Web hooks connecting to GitHub.
- Use the browser-based login, or **docker login** from the command line.

### 1.3 Starting Containers

#### Starting Containers

- Many environments are available, let's look at Docker.
- As Docker is no loginer supported on Red Hat, we will use Docker on Ubuntu.
- To use containers on RHEL 8 and related distributions, use **podman** instead of **docker**
- Use **dnf install -y container-tools** to install **podman** and related utilities.

#### Demo: Installing Docker on Ubuntu

```bash
mkdir -p ~/bin && cd $_
curl -fsSL https://get.docker.com -o get-docker.sh;
sudo sh get-docker.sh;
sudo usermod -aG docker ${USER};
newgrp docker;
docker run hello-world
```

#### Demo: Running a Container

```bash
mkdir -p /var/www/html
echo "Hello from Docker!" >> /var/www/html/index.html
cat /var/www/html/index.html
docker run -d -p 8080:80 --name="myapache" -v /var/www/html:/var/www/html httpd
docker ps
ss -tunap | grep 8080
curl http://localhost:8080
```

#### Running Another Container

- **docker run -it busybox** will start the busybox container with an interactive terminal to the entrypoint application.
- **Ctrl-p, Ctrl-q** to disconnect and keep it running.
- **exit** to stop the current container application.
  - If you were connected to the entrypoint application, the container will stop.
  - If you were connected to another shell session, the container will continue.

### 1.4 Managing Containers

#### Common Commands Overview

- **docker ps [-a]**: shows currently [and paste] running containers.
- **docker start**: starts a container from a locally stored image.
- **docker stop**: stops a container using Linux SIGTERM
- **docker restart**: restarts a currently runing container.
- **docker kill**: stops a container using Linux SIGKILL
- **docker run**: removes all container files from the host operating system.

#### Inspecting Container Settings

- **docker ps** will show the IDs fo containers, pick one.
- **docker inspect <ID> | less**
- docker inspect --format='{{.NetworkSettings.IPAddress}}' containername
- docker inspect --format='{{.State.Pid}}' containername
- Alternatively, use **ps aux** on the host to find the container PID.

### 1.5 Managing Container Images

#### Managing Images

- After fetching, the container images are stored on the host.
- Use **docker images** to get a list of images
- Or use **docker image --help** to get an overview of all options related to managing container images.
```bash
docker images;
docker image ls;
docker image inspect <IMAGE ID>
docker image rm -f <IMAGE ID>
```

### 1.6 Understanding Container Logging

#### Understanding Container Logging

- The container application does not connect to a STDOUT, which is why logs, by default, are written to the container.
- Use **docker logs mycontainer** to get access to the container log.
- Using **docker logs** is convenient for troubleshooting.
```bash
docker run -d mariadb;
docker ps;
docker ps -a;
docker logs <containername>
docker run --name mydb -e MYSQL_ROOT_PASSWORD=password mariadb
docker ps;
```

### Lesson 1 Lab Using Containers

- Install Docker on your test environment.
- Run the latest version of Fedora in a container.
- In interactive mode, start a bash shell and explore the /etc/os-release file, as well as the kernel version (**uname -r**)
- Disconnect from the container without shutting it down.

### Lesson 1 Lab Solution Using Containers

- Follow the docs above to install the Docker.
```bash
docker run -it ubuntu:latest
cat /etc/os-release
uname -r
docker ps;
```
