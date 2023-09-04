# Lesson 2 - Managing Container Images

- [Lesson 2 - Managing Container Images](#lesson-2---managing-container-images)
    - [2.1 Understanding Image Architecture](#21-understanding-image-architecture)
      - [Understanding Image Architecture](#understanding-image-architecture)
    - [2.2 Tagging Container Images](#22-tagging-container-images)
      - [Exploring Image Layers](#exploring-image-layers)
      - [Tagging Images](#tagging-images)
    - [2.3 Understanding Image Creation Options](#23-understanding-image-creation-options)
      - [Creating Images](#creating-images)
    - [2.4 Building Custom Container Images](#24-building-custom-container-images)
      - [Options for Customizing Images](#options-for-customizing-images)
      - [About Terminology](#about-terminology)
      - [Understanding Parent/Child Images](#understanding-parentchild-images)
      - [Understanding the Process](#understanding-the-process)
    - [2.5 Using Dockerfile/Containerfile](#25-using-dockerfilecontainerfile)
      - [About Terminology](#about-terminology-1)
      - [Writing a Dockerfile](#writing-a-dockerfile)
      - [Understanding ENTRYPOINT](#understanding-entrypoint)
      - [Understanding Formats](#understanding-formats)
      - [Avoiding Multi-layer Images](#avoiding-multi-layer-images)
      - [Building Images with Dockerfile](#building-images-with-dockerfile)
    - [2.6 Creating Images with Docker Commit](#26-creating-images-with-docker-commit)
      - [Using docker commit](#using-docker-commit)
    - [Lesson 2 Lab Creating Custom Container Images](#lesson-2-lab-creating-custom-container-images)
    - [Lesson 2 Lab Solution Creating Custom Container Images](#lesson-2-lab-solution-creating-custom-container-images)

### 2.1 Understanding Image Architecture

#### Understanding Image Architecture

- A container image basically is a tar file with associated metadata.
- To build container images in an efficient way, it typically consists of multiple layers.
- While building an image, a base system image is used.
- On top of the base system image, the application is installed as an additional layer.
- Some of the standard images themselves alredy consist of multiple layers.
- Using layers ensures that images are built efficiently.

### 2.2 Tagging Container Images

#### Exploring Image Layers

- **docker image ls** shows images stored, including version information using tags.
- **docker image rm** removes images
- **docker history <ImageID>** or **docker history image:tag** shows the different layers in the image.
- Notice that each modification adds an image layer.

#### Tagging Images

- Tags allow you to assign names to images, which makes it easier to manage versions.
  - If no tag is added, "latest" is used as the default tag.
- Manually tag images: **docker tag myapache myapache:1.0**
  - Next, using **docker images ls | grep myapache** will show the same image listed twice, as 1.0 and as latest.
- Tags can also be used to identify the target registry.
  - **docker tag myapache localhost:5000/myapache:1.0**
```bash
docker image ls;
docker tag mariadb localhost:5000/mariadb:latest
docker image ls;
docker image --help
docker image history mariadb:latest;
```

### 2.3 Understanding Image Creation Options

#### Creating Images

There are two main approaches to creating an image.
- Using a running container: a container is started, and modification are applied to the container. The **docker commit** command is used to write modifications.
- Using a Dockerfile: a Dockerfile contains instuctions for building an image. Each instructions adds a new layer to the image, which offers more control over the files that are added to an image at a later stage. 

### 2.4 Building Custom Container Images

#### Options for Customizing Images

- Dockerfile, also known as Containerfile, is a very common way to build custom images.
- It uses a descriptive file using a base image in which commands are executed to customize it.
- **docker commit** commits changes that are made in a running container to the container image, after which **docker save** can be used to save it.
- **buildah** can be used to create, modify and manage container images.

#### About Terminology

- Dockerfile is the original terminology, which was introduced by Docker.
- OCI has standardized the name to Containerfile.
- Dockerfile should be used while working with **docker**, Containerfile may be used while working with **podman** 

#### Understanding Parent/Child Images

- When working with Dockerfile, using Parent/Child images is common.
- A child image is an image that is created from a parent image and incorporates everything in the parent image.
- Starting from a parent image makes it easier to create a reliable image.

#### Understanding the Process

- First, you will create a working directory: each project should have its own project directory.
- Next, you will write the Dockerfile: the Dockerfile contains instructions to build the image.
- The most important part in the Dockerfile is the default command it will start, which is done by using ENTRYPOINT or CMD.
- Finally, build the image with **docker build** or **podman build**

### 2.5 Using Dockerfile/Containerfile

#### About Terminology

- Dockerfile was introduced by Docker to automate building container images.
- In Red Hat environments, the word Containerfile is preferred.
- Currently, **docker** only supports Dockerfile and **podman** works with Dockerfile as well as Containerfile.
- For this reason, in this lesson I'm using the word Dockerfile.

#### Writing a Dockerfile

- Each Dockerfile starts with **FROM**, identifying the base image to use.
  - Next, instructions are executed in that base image.
  - Instructions are executed in the order specified.
- Each Dockerfile instruction runs in an independent container, using an intermediate image built from a previous command, which means that adding multiple instructions results in multiple layers.
- Having multiple layers makes the image less efficient.

#### Understanding ENTRYPOINT

- **ENTRYPOINT** is the default command to be processed.
- If not specified, **/bin/sh -c** is executed as the default command.
- Arguments to the **ENTRYPOINT** command may be specified separately using **CMD**
  - **ENTRYPOINT** ["command"]; **ENTRYPOINT ["/usr/sbin/httpd"]**
  - **CMD** ["arg1", "arg2"]; **CMD ["-D", "FOREGROUND"]**
- If the default command is specified using **CMD** instead of **ENTRYPOINT**, the command is executed as an argument to the default entrypoint **sh -c** which can give unexpected results.
- If the arguments to the command are specified within the **ENTRYPOINT**, then they are not supposed to be overwritten from the command line.

#### Understanding Formats

- Options like **ADD, COPY, ENTRYPOINT, CMD** are used in shell form and in exec form.
- Shell form is a list of items.
  - **ADD ["/my/file", "/mydir"]**
  - **ENTRYPOINT ["/usr/bin/nmap", "-sn", "172.17.0.0/24"]**
- Using Exec form is preferred, as shell form wraps command in a **/bin/sh -c** shell, which creates a sometimes unnecessary shell process.

#### Avoiding Multi-layer Images

- Each command used in a Dockerfile creates a new layer and this should be avoided.
- So don't run multiple **RUN** commands, connect them using **&&**
  - **RUN yum --disablerepo=* --enablerepo="myrepo" && yum update -y && yum install nmap**
  - To maintain readability, write the commands on different lines using **&& \** at the end of each line.
```dockerfile
RUN yum --disablerepo=* --enablerepo="rhel7-server-rpms" && \
    yum update -y && \
    yum install -y nginx
```

#### Building Images with Dockerfile

- To build an image with Dockerfile, use **docker build** / **podman build**
- **docker build** takes 2 arguments: **-t name[:tag] directory**
- If no tag is specified, the image is tagged as latest.
  - **docker build -t myimage .**
```bash
cd labs/dockerfile/
vim Dockerfile
docker build -t nmap .
docker images;
docker build -t nmap . --no-cache
```

### 2.6 Creating Images with Docker Commit

#### Using docker commit

- After making changes to a container, you can save it to an image.
- Use **docker commit** to do so
  - **docker commit -m "custom web server" -a "Anil Kumar N" myapache myapache**
  - Use **docker images** to verify
- Next, use **docker save -o myapache.tar myapache** and transport it to anywhere you'd like.
- From another system, use **docker load -i myapache.tar** to import it as an image.
```bash
docker exec -it myapache sh
echo hello > /tmp/hellofile
ls -l /tmp/hellofile
exit
docker diff myapache
docker commit --help
docker commit myapache myapache
docker images
docker save -o myapache.tar myapache
ls -l
```

### Lesson 2 Lab Creating Custom Container Images

- Create a Dockerfile that creates an image that meets the following requirements.
  - Based on Fedora.
  - Contains the packages containing the **ps** command as well as network tools.
  - Should run the sshd process

### Lesson 2 Lab Solution Creating Custom Container Images

```bash
cd labs/lesson1-labs/
vim Dockerfile
docker build -t mysshd .
docker images
docker run mysshd
```
