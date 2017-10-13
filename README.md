Kubernetes Init Containers Update Issue
=======================================

I've discovered what appears to be an issue with updating a pod/deployment specification's init containers. This issue does not exist when using the legacy approach for defining init containers (using the `pod.beta.kubernetes.io/init-containers` metadata annotation), but does appear when using the latest approach (`initContainers` section in the pod spec).

This project contains the necessary artifacts for recreating the issue using minikube.

I initially ran into this problem on Google Container Engine running Kubernetes v1.7.5.

Steps for recreating the issue using minikube
---------------------------------------------

*This guide assumes you already have minikube installed locally!*

Clone this repository and then run these initial commands:

```
minikube start
eval $(minikube docker-env)
docker build -t hello-world-init:1 -f Dockerfile.initContainer .
docker build -t hello-world-nginx:1 -f Dockerfile.container .
docker build -t hello-world-init:2 -f Dockerfile.initContainer .
docker build -t hello-world-nginx:2 -f Dockerfile.container .
```

### Legacy annotation configuration approach (*WORKS CORRECTLY*)

Apply the k8s configuration that uses the legacy annotation for defining init containers:

```
kubectl apply -f hello-world.annotation.yaml`
```

Now edit `hello-world.annotation.yaml` and update the image tags for the two referenced containers:

```
hello-world-init:1 => hello-world-init:2
hello-world-nginx:1 => hello-world-nginx:2
```

Save changes and the re-apply the configuration:

```
kubectl apply -f hello-world.annotation.yaml`
```

Inspect the deployment:

```
kubectl describe deployment hello-world-deployment
```

And you should see that the both the `hello-world-init` container and the `hello-world-nginx` container are running version 2 of their respective images. This is exactly what we'd expect to see.

### Latest pod spec configuration approach (*WORKS INCORRECTLY*)

Firstly clean up the previous resources:

```
kubectl delete deployment hello-world-deployment
kubectl delete service hello-world-service
```

Apply the k8s configuration that uses the pod spec for defining init containers:

```
kubectl apply -f hello-world.spec.yaml`
```

Now edit `hello-world.spec.yaml` and update the image tags for the two referenced containers:

```
hello-world-init:1 => hello-world-init:2
hello-world-nginx:1 => hello-world-nginx:2
```

Save changes and the re-apply the configuration:

```
kubectl apply -f hello-world.spec.yaml`
```

Inspect the deployment:

```
kubectl describe deployment hello-world-deployment
```

And you should see that the main container `hello-world-nginx`'s image has been updated correctly to version 2, but the init container `hello-world-init`'s image remains at version 1. This does not appear to be the desired behaviour, and certainly is not consistent with the legacy annotation behaviour.
