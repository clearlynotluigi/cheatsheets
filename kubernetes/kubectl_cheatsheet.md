# Kubectl Power Cheatsheet 🕹️

> A practical, beautiful guide to the most common `kubectl` commands. Your essential reference for interacting with any Kubernetes cluster, designed for clarity and easy copy-pasting.

**Basic Command Structure:** `kubectl <command> <resource-type> <resource-name> [flags]`

### 🔗 Managing Your Connection

*Control which cluster you're talking to and where you're working inside it.*

#### `kubectl config get-contexts`

*Lists all your saved connections to clusters.*

> The active context is marked with an asterisk (`*`). This is the first command to run if you're not sure which cluster you're connected to.

```shell
kubectl config get-contexts
```

#### `kubectl config use-context <name>`

*Switches your active session to a different cluster connection.*

```shell
# Example: Switch your command line to talk to the minikube cluster
kubectl config use-context minikube
```

#### `kubectl get namespaces`

*Lists all the "virtual partitions" (namespaces) inside your cluster.*

> **Pro Tip:** Use the shorthand `ns`. `kubectl get ns` is much faster to type!

```shell
kubectl get ns
```

#### `kubectl create namespace <name>`

*Creates a new, empty namespace for you to organize your work.*

```shell
# Example: Create a namespace called 'dev' for development work
kubectl create namespace dev
```

### 🔍 Inspecting Your Cluster

*Use these safe, "read-only" commands to understand the state of your cluster.*

#### `kubectl cluster-info`

*Displays the addresses of the main Kubernetes components.*

> A great first command to run as a health check to ensure you can communicate with the cluster.

```shell
kubectl cluster-info
```

#### `kubectl get nodes`

*Shows the machines (nodes) that make up your cluster and their status.*

```shell
# Use the -o wide flag to see more info like IP addresses and OS
kubectl get nodes -o wide
```

#### `kubectl get all`

*A fantastic summary of all major resources running in a specific namespace.*

```shell
# Get all resources in the 'dev' namespace
kubectl get all -n dev
```

### 📦 Working with Pods

*A Pod is the smallest deployable unit in Kubernetes, wrapping one or more containers.*

#### `kubectl run <name> --image=<image>`

*Quickly creates a single Pod running a specific container image.*

> This is great for quick tests, but for real applications, always use a Deployment.

```shell
# Run a single Nginx pod in the 'dev' namespace
kubectl run nginx-pod --image=nginx -n dev
```

#### `kubectl describe pod <name>` 🌟

*Shows detailed information and recent events about a specific Pod.*

> **This is your #1 debugging tool!** If a Pod is stuck or failing, the `Events` section at the bottom of the output will almost always tell you why.

```shell
kubectl describe pod nginx-pod -n dev
```

#### `kubectl logs <pod-name>`

*Streams the logs from the container inside the Pod.*

```shell
# Use the -f flag to "follow" the logs in real-time
kubectl logs -f nginx-pod -n dev
```

#### `kubectl exec -it <pod-name> -- <command>`

*Run a command *inside* the container of a running Pod.*

> `it` stands for "interactive terminal", which you need for shells.

```shell
# Get an interactive bash shell inside the nginx-pod
kubectl exec -it nginx-pod -n dev -- /bin/bash
```

### 🚀 Managing Deployments

*A Deployment manages a set of identical Pods, ensuring your application is self-healing and scalable.*

#### `kubectl create deployment <name> --image=<image>`

*Creates a Deployment, which in turn creates and manages Pods for you.*

```shell
# Create a deployment named 'my-nginx' with 1 replica pod
kubectl create deployment my-nginx --image=nginx -n dev
```

#### `kubectl scale deployment <name> --replicas=<count>`

*Changes the number of running Pods managed by the Deployment.*

> This is how you scale your application up or down instantly.

```shell
# Scale the my-nginx deployment to 3 pods for high availability
kubectl scale deployment my-nginx --replicas=3 -n dev
```

#### `kubectl delete deployment <name>`

*Deletes the Deployment and all the Pods it was managing.*

```shell
kubectl delete deployment my-nginx -n dev
```

### 🌐 Accessing Your Application

*Expose your running application to the network.*

#### `kubectl expose deployment <name>`

*Creates a "Service," which is a stable network address for your set of Pods.*

```shell
# Expose the 'my-nginx' deployment on port 80 as a NodePort service
kubectl expose deployment my-nginx --port=80 --type=NodePort -n dev
```

#### `kubectl get services`

*Lists all the Services in your namespace.*

> Use the shorthand `svc` to save time.

```shell
kubectl get svc -n dev
```

#### `kubectl port-forward <resource>/<name> <local-port>:<remote-port>`

*Creates a direct, temporary tunnel from your Mac into a Pod or Service.*

> This is extremely useful for development and debugging without needing a full Service.

```shell
# Forward your local port 8080 to port 80 on the my-nginx service.
# Now visit http://localhost:8080 in your browser!
kubectl port-forward svc/my-nginx 8080:80 -n dev
```

#### `minikube service <service-name>`

*A handy **Minikube-specific** shortcut that opens your Service directly in a web browser.*

```shell
minikube service my-nginx -n dev
```