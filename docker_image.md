# Building and running a docker image

These are the instructions for the third part of the workshop where you will be using Docker.

## Pre-requisites

 - Docker already installed on your local machine.
 - An account created in [Docker Hub](https://hub.docker.com).

## Lab steps

### Create a Python app (without using Docker)

Run the following command to create a file named app.py with a simple python program. (Copy and paste the entire code block.)
```
echo 'from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "hello world!"

if __name__ == "__main__":
    app.run(host="0.0.0.0")' > app.py

```
This is a simple python app that uses Flask to expose an HTTP web server on port 5000. (5000 is the default port for flask.) Don’t worry if you are not too familiar with Python or Flask. These concepts can be applied to an application written in any language.
Optional: If you have Python and pip installed, you can locally run this app. If not, move on to the next step.

### Create and build the Docker image

1.	Create a file named Dockerfile and add the following content:

```
FROM python:3.6.1-alpine
RUN pip install flask
CMD ["python","app.py"]
COPY app.py /app.py
```

2.	Build the Docker image.
Pass in -t to name your image python-hello-world.

```
$ docker image build -t python-hello-world .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM python:3.6.1-alpine
3.6.1-alpine: Pulling from library/python
acb474fa8956: Pull complete 
967ab02d1ea4: Pull complete 
640064d26350: Pull complete 
db0225fcac8f: Pull complete 
5432cc692c60: Pull complete 
Digest: sha256:768360b3fad01adffcf5ad9eccb4aa3ccc83bb0ed341bbdc45951e89335082ce
Status: Downloaded newer image for python:3.6.1-alpine
 ---> c86415c03c37
Step 2/4 : RUN pip install flask
 ---> Running in cac3222673a3
Collecting flask
  Downloading Flask-0.12.2-py2.py3-none-any.whl (83kB)
Collecting itsdangerous>=0.21 (from flask)
  Downloading itsdangerous-0.24.tar.gz (46kB)
Collecting click>=2.0 (from flask)
  Downloading click-6.7-py2.py3-none-any.whl (71kB)
Collecting Werkzeug>=0.7 (from flask)
  Downloading Werkzeug-0.12.2-py2.py3-none-any.whl (312kB)
Collecting Jinja2>=2.4 (from flask)
  Downloading Jinja2-2.9.6-py2.py3-none-any.whl (340kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->flask)
  Downloading MarkupSafe-1.0.tar.gz
Building wheels for collected packages: itsdangerous, MarkupSafe
  Running setup.py bdist_wheel for itsdangerous: started
  Running setup.py bdist_wheel for itsdangerous: finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/fc/a8/66/24d655233c757e178d45dea2de22a04c6d92766abfb741129a
  Running setup.py bdist_wheel for MarkupSafe: started
  Running setup.py bdist_wheel for MarkupSafe: finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/88/a7/30/e39a54a87bcbe25308fa3ca64e8ddc75d9b3e5afa21ee32d57
Successfully built itsdangerous MarkupSafe
Installing collected packages: itsdangerous, click, Werkzeug, MarkupSafe, Jinja2, flask
Successfully installed Jinja2-2.9.6 MarkupSafe-1.0 Werkzeug-0.12.2 click-6.7 flask-0.12.2 itsdangerous-0.24
 ---> ce41f2517c16
Removing intermediate container cac3222673a3
Step 3/4 : CMD python app.py
 ---> Running in 2197e5263eff
 ---> 0ab91286958b
Removing intermediate container 2197e5263eff
Step 4/4 : COPY app.py /app.py
 ---> f1b2781b3111
Removing intermediate container b92b506ee093
Successfully built f1b2781b3111
Successfully tagged python-hello-world:latest
```

3.	Run docker image ls to verify that your image shows up in your image list.

```
$ docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
python-hello-world   latest              f1b2781b3111        26 seconds ago      99.3MB
python               3.6.1-alpine        c86415c03c37        8 days ago          88.7MB
```

Notice that your base image, python:3.6.1-alpine, is also in your list.

### Run the Docker image

Now that you have built the image, you can run it to see that it works.

1.	Run the Docker image.
```
$ docker run -p 5001:5000 -d python-hello-world
0b2ba61df37fb4038d9ae5d145740c63c2c211ae2729fc27dc01b82b5aaafa26
```
-p flag maps a port running inside the container to your host. In this case, you’re mapping the Python app running on port 5000 inside the container to port 5001 on your host. Note that if port 5001 is already in use by another application on your host, you may have to replace 5001 with another value, such as 5002.

2.	Navigate to http://localhost:5001 in a browser to see the results.
You should see “hello world!” on your browser.

3.	Check the log output of the container.
If you want to see logs from your application you can use the docker container logs command. By default, docker container logs prints out what is sent to standard out by your application. Use docker container ls to find the id for your running container.
```
$ docker container logs [container id] 
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
172.17.0.1 - - [28/Jun/2017 19:35:33] "GET / HTTP/1.1" 200 -
```

## Push to a central registry

1.	Navigate to [Docker hub](https://hub.docker.com) and create a free account, if you haven’t already.

2.	Log in to the Docker registry account by typing docker login on your terminal.
```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: 
```

3.	Tag your image with your username.
```
$ docker tag python-hello-world myusername/python-hello-world:v1
```

4.	Once you have a properly tagged image, use the docker push command to push your image to the Docker Hub registry.
```
$ docker push myusername/python-hello-world:v1
The push refers to a repository [docker.io/myusername/python-hello-world:v1]
2bce026769ac: Pushed 
64d445ecbe93: Pushed 
18b27eac38a1: Mounted from library/python 
3f6f25cd8b1e: Mounted from library/python 
b7af9d602a0f: Mounted from library/python 
ed06208397d5: Mounted from library/python 
5accac14015f: Mounted from library/python 
latest: digest: sha256:508238f264616bf7bf962019d1a3826f8487ed6a48b80bf41fd3996c7175fd0f size: 1786
```

5.	Check out your image on docker hub in your browser.
Navigate to [Docker hub](https://hub.docker.com), and go to your profile to see your newly uploaded image.
