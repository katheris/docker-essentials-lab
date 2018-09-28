# Running containers in production

These are the instructions for the third part of the workshop where you will start using Kubernetes.

## Pre-requisites

 - A Docker container to use in Kubernetes, the docs assume this is the one used in previous parts of the workshop

## Installing Kubernetes

### Mac

If you have [Docker for Mac](https://docs.docker.com/docker-for-mac/) you can use Kubernetes that comes built in.
 - Click on the Docker icon in the top bar of your screen and select Preferences.
 - Click on the Kubernetes tab, enable it and choose Kubernetes, rather than Swarm.

See the [docs](https://docs.docker.com/v17.09/docker-for-mac/#kubernetes) for more details.

### Linux

On Linux there are multiple options for running Kubernetes. A simple local environment is provided by [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

### Windows

If you have [Docker for Windows](https://docs.docker.com/docker-for-windows) you can use the built in Kubernetes.
 - Open the Docker settings and click the Kubernetes tab
 - Click "Enable Kubernetes"

See the [docs](https://docs.docker.com/docker-for-windows/#kubernetes) for more details.

## Kubernetes CLI

Kubernetes has a CLI called `kubectl` which should be already installed with Kubernetes.

To check your connection run:

```kubectl cluster-info```

This should give the following response:

```

Kubernetes master is running at https://localhost:6443
KubeDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```

If you don't get the above try switching the Kubernetes context to `docker-for-desktop`:

 - Click on the Docker icon in the top bar of your screen and select Kubernetes
 - Select `docker-for-desktop`

You can also use `kubectl` to check the status of the nodes that Kubernetes has available:

```
$ kubectl get nodes

NAME                 STATUS    ROLES     AGE       VERSION
docker-for-desktop   Ready     master    7m        v1.10.3
```

And to see the services that are currently running: (by default this will just be the Kubernetes service)

```
$ kubectl get services

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10m
```

## Try out Kubernetes

### Create a deployment

First we will create a deployment to stand up the Python app.

We will use the `deploy.yaml` in the `orchestration` folder of this repo:

```
$ kubectl create -f deploy.yaml
deployment "hello-deploy" created
```

See the deployment running and the pod it has created:

```
$ kubectl get deployment
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-deploy   1         1         1            1           28s
```

```
$ kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
hello-deploy-7478bb6b9f-vjpd6   1/1       Running   0          1m
```

## Scale the deployment

In one shell window watch the pods:

```
$ kubectl get pods -w
```

In another update the deployment to increase the number of replicas to three:

```
$ kubectl edit deployment hello-deploy
```

 - Type `i` to start editing
 - Change `replicas` from `1` to `3`
 - To save the changes hit the `esc` key and then type `:`, then `wq` and press `Enter`

You should see the message:
```
deployment "hello-deploy" edited
```

In the first window you should have seen more pods being created:
```
$ kubectl get pods -w
NAME                            READY     STATUS    RESTARTS   AGE
hello-deploy-7478bb6b9f-vjpd6   1/1       Running   0          2m
hello-deploy-7478bb6b9f-dtn6q   0/1       Pending   0         0s
hello-deploy-7478bb6b9f-5dt9n   0/1       Pending   0         0s
hello-deploy-7478bb6b9f-dtn6q   0/1       Pending   0         0s
hello-deploy-7478bb6b9f-5dt9n   0/1       Pending   0         0s
hello-deploy-7478bb6b9f-dtn6q   0/1       ContainerCreating   0         0s
hello-deploy-7478bb6b9f-5dt9n   0/1       ContainerCreating   0         0s
hello-deploy-7478bb6b9f-5dt9n   1/1       Running   0         3s
hello-deploy-7478bb6b9f-dtn6q   1/1       Running   0         3s
```

There should now be three pods:
```
$ kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
hello-deploy-7478bb6b9f-5dt9n   1/1       Running   0          4m
hello-deploy-7478bb6b9f-dtn6q   1/1       Running   0          4m
hello-deploy-7478bb6b9f-vjpd6   1/1       Running   0          12m
```

## Kubernetes self-healing

In a shell window watch the pods with `kubectl get pods -w`

In a second window delete one of the pods:
```
$ kubectl delete pod hello-deploy-7478bb6b9f-5dt9n
pod "hello-deploy-7478bb6b9f-5dt9n" deleted
```

In the first window watch Kubernetes self-healing:

```
$ kubectl get pods -w
NAME                            READY     STATUS    RESTARTS   AGE
hello-deploy-7478bb6b9f-5dt9n   1/1       Running   0          7m
hello-deploy-7478bb6b9f-dtn6q   1/1       Running   0          7m
hello-deploy-7478bb6b9f-vjpd6   1/1       Running   0          15m
hello-deploy-7478bb6b9f-5dt9n   1/1       Terminating   0         7m
hello-deploy-7478bb6b9f-lzfd2   0/1       Pending   0         0s
hello-deploy-7478bb6b9f-lzfd2   0/1       Pending   0         0s
hello-deploy-7478bb6b9f-lzfd2   0/1       ContainerCreating   0         0s
hello-deploy-7478bb6b9f-lzfd2   1/1       Running   0         2s
```

You should now have three pods again:
```
$ kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
hello-deploy-7478bb6b9f-dtn6q   1/1       Running   0          9m
hello-deploy-7478bb6b9f-lzfd2   1/1       Running   0          1m
hello-deploy-7478bb6b9f-vjpd6   1/1       Running   0          17m
```

## Create a Kubernetes service

Create a service with a NodePort to make the pods addressable:

```
$ kubectl apply -f service.yaml
service "hello-service" created

$ kubectl get services
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hello-service   NodePort    10.111.244.203   <none>        8080:30000/TCP   15s
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          38m
```

You should now be able to curl 30000 to talk to the app:
```
$ curl http://localhost:30000
```

The service is routing you to one of the instances.

## Deploying multiple versions

We will now deploy a second version to see how the service works.

First scale the existing deployment back down to 1 replica with `kubectl edit deployment hello-deploy`

Make sure you only have one pod running:
```
$ kubectl get pods
NAME                            READY     STATUS        RESTARTS   AGE
hello-deploy-7478bb6b9f-vjpd6   1/1       Running       0          23m
```

Update your Python app to give a different message and rebuild it:

```
$ docker build -t devcamp/hello-world:v2 .
```

Update the `deploy.yaml` with the image name `devcamp/hello-world:v2` and give the deployment a different name.

Create the deployment:
```
$ kubectl apply -f deploy.yaml 
deployment "hello-deploy-v2" created
```

You should now have 2 deployments:
```
$ kubectl get deployments
NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-deploy      1         1         1            1           27m
hello-deploy-v2   1         1         1            1           13s
```

and 2 pods:

```
$ kubectl get pods
NAME                               READY     STATUS    RESTARTS   AGE
hello-deploy-7478bb6b9f-vjpd6      1/1       Running   0          27m
hello-deploy-v2-5c495f6669-ltrvx   1/1       Running   0          38s
```

Try running the curl command multiple times and see it calling different versions of the app:

```
$ curl http://localhost:30000
Hello World !
$ curl http://localhost:30000
Hello World !
$ curl http://localhost:30000
Hello World Version 2 !
$ curl http://localhost:30000
Hello World Version 2 !
$ curl http://localhost:30000
Hello World Version 2 !
$ curl http://localhost:30000
Hello World !
$ curl http://localhost:30000
Hello World !
```
