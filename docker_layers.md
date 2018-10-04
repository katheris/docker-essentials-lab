# Updating your docker image

These are the instructions for the third part of the workshop where you will be using Docker.

## Pre-requisites

This docs assumes that you have already built a docker image and pushed it to the Docker Hub registry following the instructions in [Building and running a docker image](docker_image.md). 

## Lab steps

### Deploy a change

1.	Update *app.py* by replacing the string **Hello World!** with **Hello Beautiful World!**. 
Your file should have the following contents:
```
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello Beautiful World!"


if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

2.	Now that your app is updated, you need to rebuild your app and push it to the [Docker Hub](https://hub.docker.com) registry.
First rebuild, this time use your Docker Hub username in the build command.
```
$  docker image build -t myusername/python-hello-world:v2 .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM python:3.6.1-alpine
 ---> c86415c03c37
Step 2/4 : RUN pip install flask
 ---> Using cache
 ---> ce41f2517c16
Step 3/4 : CMD python app.py
 ---> Using cache
 ---> 0ab91286958b
Step 4/4 : COPY app.py /app.py
 ---> 3e08b2eeace1
Removing intermediate container 23a955e881fc
Successfully built 3e08b2eeace1
Successfully tagged jzaccone/python-hello-world:v2
```

Notice the “Using cache” for Steps 1-3. These layers of the Docker image have already been built, and docker image build will use these layers from the cache, instead of rebuilding them.
```
$ docker push myusername/python-hello-world:v2
The push refers to a repository [docker.io/myusername/python-hello-world:v2]
94525867566e: Pushed 
64d445ecbe93: Layer already exists 
18b27eac38a1: Layer already exists 
3f6f25cd8b1e: Layer already exists 
b7af9d602a0f: Layer already exists 
ed06208397d5: Layer already exists 
5accac14015f: Layer already exists 
latest: digest: sha256:91874e88c14f217b4cab1dd5510da307bf7d9364bd39860c9cc8688573ab1a3a size: 1786
```

### Understand image layers

Image layering enables the docker caching mechanism for builds and pushes. For example, the output for your last docker push shows that some of the layers of your image already exists on the Docker Hub.
```
$ docker push myusername/python-hello-world:v2
The push refers to a repository [docker.io/jzaccone/python-hello-world:v2]
94525867566e: Pushed 
64d445ecbe93: Layer already exists 
18b27eac38a1: Layer already exists 
3f6f25cd8b1e: Layer already exists 
b7af9d602a0f: Layer already exists 
ed06208397d5: Layer already exists 
5accac14015f: Layer already exists 
latest: digest: sha256:91874e88c14f217b4cab1dd5510da307bf7d9364bd39860c9cc8688573ab1a3a size: 1786
```

To look more closely at layers, you can use the docker image history command of the python image we created.
```
$ docker image history python-hello-world
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
f1b2781b3111        5 minutes ago       /bin/sh -c #(nop) COPY file:0114358808a1bb...   159B                
0ab91286958b        5 minutes ago       /bin/sh -c #(nop)  CMD ["python" "app.py"]      0B                  
ce41f2517c16        5 minutes ago       /bin/sh -c pip install flask                    10.6MB              
c86415c03c37        8 days ago          /bin/sh -c #(nop)  CMD ["python3"]              0B                  
<missing>           8 days ago          /bin/sh -c set -ex;   apk add --no-cache -...   5.73MB              
<missing>           8 days ago          /bin/sh -c #(nop)  ENV PYTHON_PIP_VERSION=...   0B                  
<missing>           8 days ago          /bin/sh -c cd /usr/local/bin  && ln -s idl...   32B                 
<missing>           8 days ago          /bin/sh -c set -ex  && apk add --no-cache ...   77.5MB              
<missing>           8 days ago          /bin/sh -c #(nop)  ENV PYTHON_VERSION=3.6.1     0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV GPG_KEY=0D96DF4D411...   0B                  
<missing>           8 days ago          /bin/sh -c apk add --no-cache ca-certificates   618kB               
<missing>           8 days ago          /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV PATH=/usr/local/bin...   0B                  
<missing>           9 days ago          /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B                  
<missing>           9 days ago          /bin/sh -c #(nop) ADD file:cf1b74f7af8abcf...   4.81MB  
```

Each line represents a layer of the image. You’ll notice that the top lines match to your Dockerfile that you created, and the lines below are pulled from the parent python image. Don’t worry about the `<missing>`tags. These are still normal layers; they have just not been given an ID by the docker system.

### Clean up

Completing this lab results in a lot of running containers on your host. Let’s clean these up.
1.	Get a list of the containers running using docker container ls.
```
$ docker container ls
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                    NAMES
0b2ba61df37f        python-hello-world   "python app.py"     7 minutes ago       Up 7 minutes        0.0.0.0:5001->5000/tcp   practical_kirch
```

2.	Run docker container stop [container id] for each container in the list that is running.
```
$ docker container stop 0b20b2
```

3.	Remove the stopped containers.
docker system prune is a really handy command to clean up your system. It will remove any stopped containers, unused volumes and networks, and dangling images.
```
$ docker system prune
WARNING! This will remove:
        - all stopped containers
        - all volumes not used by at least one container
        - all networks not used by at least one container
        - all dangling images
Are you sure you want to continue? [y/N] y
Deleted Containers:
0b2ba61df37fb4038d9ae5d145740c63c2c211ae2729fc27dc01b82b5aaafa26

Total reclaimed space: 300.3kB
```



