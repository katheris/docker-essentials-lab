# Running containers in production

These are the instructions for the third part of the workshop where you will start using Kubernetes.

## Pre-requisites

 - A Docker container to use in Kubernetes, the docs assume this is the one used in previous parts of the workshop

## Installing Kubernetes

### Mac

If you have [Docker for Mac](https://docs.docker.com/docker-for-mac/) you can use Kubernetes that comes built in.
 - Click on the Docker icon in the top bar of your screen and select Preferences.
 - Click on the Kubernetes tab, enable it and choose Kubernetes, rather than Swarm.

### Linux

TODO

### Windows

TODO

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

## Try out Kubernetes


