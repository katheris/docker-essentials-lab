# Building and running a docker image

These are the instructions for the first part of the Docker workshop.

## Pre-requisites

 - Laptop with Windows, Linux or Mac
 - Terminal

## Lab steps

### Install Docker CE for your OS

 - [Docker CE for Windows](https://docs.docker.com/docker-for-windows/install/)
 - [Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
 - [Docker CE for Mac](https://docs.docker.com/docker-for-mac/install/)

Run the Hello World container to check that Docker is working properly. Type in your command line:

```
docker run hello-world
```
This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.

Lets see what happened.

Docker pulled the hello-world image from Docker Hub to your local registry and it stood up the hello-world container and then immediately exited it.

Check what you have in your local registry to thee the hello-world image. Type in your command line:

```
docker images
```


You should see:
```
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
hello-world                          latest              4ab4c602aa5e        3 weeks ago         1.84kB
```

Lets get an Ubuntu image and run it inside a container:
```
docker pull ubuntu
```

Check your registry now. Type in your command line:

```
docker images
```

You should see

```
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
hello-world                          latest              4ab4c602aa5e        3 weeks ago         1.84kB
ubuntu                               latest              cd6d8154f1e1        4 weeks ago         84.1MB
```

Run the Docker container from the image you just pulled:

```
docker run -i -t ubuntu bash
```

Open a new terminal and type docker ps to see your running containers

```
docker ps
```

You should see:
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b53f3fa59ef8        ubuntu              "bash"              7 seconds ago       Up 7 seconds
```

Move to the first terminal where you're inside the container and type:

```
exit
```

The container will shut done when you've exited it.


If you have some data in your container that you wanted to keep after the container is shut down, you need a way for the data to persist outside the container. For this, we use volumes. The volume will get created automatically.

When you run your container, you need to mount the volume that maps from your local filesystem to inside your Docker container. That way, when the container is destroyed, the data is still available and can be mounted to a new Docker container.

Run your ubuntu container and map a local file folder to it:

```
docker run -i --volume ~/my_laptop_folder:/home/my_docker_folder -t ubuntu bash
```
This is the end of this part of the workshop. 
