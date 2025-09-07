# Minikube & Podman Cheatsheet 🚀

> Your complete guide to a powerful local Kubernetes workflow on a Mac using Minikube with the Podman driver.

---
### 🏁 Basic Cluster Lifecycle
---
*Manage the state of your Minikube "virtual machine" (which is actually a container).*

#### `minikube start`
*Starts the cluster.*
> Creates a new one if it doesn't exist, or restarts a stopped one.

#### `minikube stop`
*✅ (Safe Stop) Stops the cluster container without deleting it.*
> Frees up RAM/CPU. This is the recommended way to pause your work.

#### `minikube pause`
*Pauses all Kubernetes components.*
> Less common than `stop`, this freezes the cluster state.

#### `minikube unpause`
*Resumes a paused cluster.*

#### `minikube status`
*Shows the status of the cluster components (host, kubelet, apiserver).*

#### `minikube delete`
*☢️ (Destructive) Deletes the cluster container and all its data.*
> Use this when you want to start completely fresh.

#### `minikube ip`
*Gets the IP address of the Minikube cluster node.*
> This IP is essential for accessing services via `NodePort`.

---
### 🔧 Configuration & Resource Management
---
*Control how much of your Mac's resources Minikube consumes.*

#### Permanently (Global Default)
*Set the default configuration for any **new** cluster you create.*
> Use `minikube config set` to define your preferred defaults so you don't have to specify them every time.
```shell
# Set the default memory for all future clusters to 2 gigabytes
minikube config set memory 2g

# Set the default number of CPUs
minikube config set cpus 2

# To see your current configuration
minikube config view

# To unset a value and revert to the original default
minikube config unset memory
```

#### At Startup (One-Time Override)
*Specify resources when you first create a cluster.*
> This setting will be saved for this specific cluster profile until you delete it.
```shell
# Create a new cluster with a 2GB memory limit and 2 CPUs
minikube start --driver=podman --memory=2g --cpus=2
```

---
### 🏗️ Building & Loading Images with Podman
---
*How to get your locally built application images into your cluster.*

#### Workflow A: Build Directly Inside Minikube (⭐️ Highly Recommended)
*This is the fastest and most efficient method. It avoids the "load" step entirely.*
> **How it works:** You temporarily point your terminal's `podman` client to the Podman service running *inside* the Minikube VM. Any image you build is immediately available to Kubernetes.

1.  **Point your terminal to Minikube's Podman daemon:**
    ```shell
    eval $(minikube podman-env)
    ```

2.  **Build your image as usual:**
    ```shell
    # Navigate to your Dockerfile/Containerfile directory
    podman build -t my-app:1.0 .
    ```
    > The image is now instantly available inside the cluster! No loading needed.

3.  **Switch back to your local Podman daemon:**
    ```shell
    eval $(minikube podman-env -u)
    ```

#### Workflow B: Build Locally, Then Load
*Useful if you prefer to keep your local Podman environment separate.*

1.  **Build the image locally on your Mac:**
    > **Pro Tip:** Tagging with a full registry-like name (e.g., `docker.io/library/my-app`) is a robust practice that helps avoid potential bugs where Minikube gets confused by the default `localhost/` prefix.
    ```shell
    podman build -t docker.io/library/my-app:1.0 .
    ```

2.  **Load the image into Minikube by name:**
    ```shell
    minikube image load docker.io/library/my-app:1.0
    ```

#### The "Can't Fail" Method: Load from a Tarball
*This is the most reliable way if loading by name fails.*
> You save the image to a file first, then load that file. It's slower but foolproof.
```shell
# First, save your image to a .tar file
podman save -o my-app.tar docker.io/library/my-app:1.0

# Second, load that specific file into Minikube
minikube image load my-app.tar
```
---
### 🌐 Accessing Your Applications
---
*How to connect to a service running inside your cluster.*

#### `minikube service `
*Opens a browser window with a tunnel directly to your service.*
> The easiest way to quickly access a web application.

#### `minikube service  --url`
*Prints a URL (`http://127.0.0.1:PORT`) to the terminal.*
> Perfect for use with `curl` or in scripts. The command creates a temporary proxy for you.

#### `minikube tunnel`
*Creates a network route from your Mac to your cluster's `LoadBalancer` services.*
> Run this in a separate, dedicated terminal. It allows you to access services on their cluster IP.

> **Example Usage:**
```shell
# To open your 'go-hello-service' in a browser
minikube service go-hello-service

# To get a URL for scripting
SERVICE_URL=$(minikube service go-hello-service --url)
curl $SERVICE_URL
```

---
### 👀 Inspecting & Managing Workloads (`kubectl`)
---
*Checking the status of your deployments, pods, and jobs.*

#### `kubectl get all`
*See a summary of all deployments, services, pods, etc. in the current namespace.*

#### `kubectl get pods -o wide`
*See all pods, their status, and which node (IP) they are on.*

#### `kubectl describe pod ` 🌟
*Get detailed information and recent events for a specific pod.*
> **This is your #1 debugging tool!** If a pod is not starting, the `Events` section at the bottom will tell you why.

#### `kubectl logs `
*View the standard output (logs) from a pod.*

#### `kubectl logs -f `
*Follow the logs in real-time (like `tail -f`).*

#### Specifically for CronJobs
*Checking a CronJob is a two-step process: check the CronJob itself, then check the Jobs and Pods it creates.*
> A `CronJob` resource creates a `Job` resource on its schedule. The `Job` then creates a `Pod` to run your command.

1.  **Check the CronJob:**
    ```shell
    # See if your CronJob is active and its schedule
    kubectl get cronjob my-cli-cronjob

    # See the full details and any past events
    kubectl describe cronjob my-cli-cronjob
    ```

2.  **Check the Jobs it created:**
    ```shell
    # See the list of jobs created by the cronjob
    kubectl get jobs
    ```

3.  **Check the Pods created by the Job:**
    ```shell
    # Find the pod created by the latest job and check its logs
    kubectl get pods
    kubectl logs my-cli-cronjob-28789500-abcdef
    ```

---
### 🧩 Useful Add-ons
---
*Extend Minikube's functionality.*

#### `minikube addons list`
*See all available add-ons and their status.*

#### `minikube addons enable dashboard`
*Enables the Kubernetes Dashboard UI.*

#### `minikube dashboard`
*Opens the Kubernetes Dashboard in your browser.*

#### `minikube addons enable ingress`
*Enables the Ingress controller, essential for routing external traffic.*