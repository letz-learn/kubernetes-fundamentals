# Running Containers in Pods

The [Pod]() is the basic unit of compute in Kubernetes. Pods run containers - it's their job to ensure the container keeps running.

Pod specs are very simple. The minimal YAML needs some metadata, and the name of the container image to run.


## Pod YAML

This is as simple as it gets:

```
apiVersion: v1
kind: Pod
metadata:
  name: whoami
spec:
  containers:
    - name: app
      image: sixeyed/whoami:21.04
```

Every Kubernetes resource requires these four fields:

* `apiVersion` - resources are versioned to support backwards compatibility
* `kind` - the type of the object
* `metadata` - collection of additional object data
* `name` - the name of the object

The format of the `spec` field is different for every object type. For Pods, this is the minimum you need:

* `containers`- list of containers to run in the Pod
* `name` - the name of the container
* `image` - the Docker image to run

> Indentation is important in YAML - object fields are nested with spaces. 

## Run a simple Pod

Kubectl is the tool for managing objects. 

You create any object from YAML using the `apply` command - [whoami-pod.yaml](whoami-pod.yaml) is the Pod spec above:

```
kubectl apply -f labs/pods/whoami-pod.yaml
```

Or the path to the YAML file can be a web address:

```
kubectl apply -f https://TODO labs/pods/whoami-pod.yaml
```

> The output shows you that nothing has changed. Kubernetes works on **desired state** deployment

Now you can use the familiar commands to print information:

```
kubectl get pods
```

> Prints a list of Pods with basic information

```
kubectl get po -o wide
```

> Uses the short name and adds extra columns

```
kubectl describe pod whoami
```

> Prints the spec and status with an event list


## Working with Pods

In a production cluster the Pod could be running on any node. You manage it using Kubectl so you don't need access to the server directly.

Print the container logs:

```
kubectl logs whoami
```

Connect to the container inside the Pod:

```
kubectl exec -it whoami -- sh
```

> This container image doesn't have a shell installed!

Run another Pod from the spec in [sleep-pod.yaml](sleep-pod.yaml):

```
kubectl apply -f labs/pods/sleep-pod.yaml

kubectl get pods
```

> This is a container image with basic DNS tools installed

This Pod container does have a shell:

```
kubectl exec -it sleep -- sh
```

Now you're connected inside the container; you can explore the container environment:

```
hostname

whoami
```

And the Kubernetes network:

```
nslookup kubernetes

ping kubernetes
```

> The Kubernetes API server is available for Pod containers to use, but internal addresses don't support ping

## Connecting from one Pod to another

Exit the shell session on the sleep Pod:

```
exit
```

And get the IP address of the original whoami Pod:

```
kubectl get pods -o wide whoami
```

> That's the internal IP address of the Pod - any other Pod in the cluster can connect on that address

Make a request to the HTTP server in the whoami Pod from the sleep Pod:

```
kubectl exec sleep -- curl -s <whoami-pod-ip>
```

> The output is the response from the whoami server - it includesthe  hostname and IP addresses

## Lab

Pods are an abstraction over containers. They monitor the container and if it exits the Pod restarts, creating a new container. This is the first layer of high-availability Kubernetes provides.

You can see this in action - force the container in the sleep Pod to exit, and look at the details of the Pod to see what happens next.

What happens if you keep forcing the container to exit?

> Stuck? Try [hints](hints.md) or check the [solution](solution.md).
