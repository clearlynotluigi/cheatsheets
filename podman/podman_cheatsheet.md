# The Ultimate Podman Cheatsheet 🚀

> Your definitive guide to the Podman CLI. A practical, beautiful reference for managing containers, pods, images, and more, built from the complete official documentation for a daemonless, rootless workflow.

**Key Concept:** Podman is designed as a drop-in replacement for Docker. You can often alias `docker=podman` and have most commands work seamlessly.

---
### 💻 Podman Machine (for macOS & Windows)
---
*On non-Linux systems, Podman runs containers inside a small Linux VM. These commands manage that VM.*

#### `podman machine init`
*Initializes a new Podman virtual machine.*
> You typically only run this once. You can customize the VM's resources, like CPU and memory, on creation.
```shell
# Initialize the default machine with 4GB of memory and 2 CPUs
podman machine init --memory 4096 --cpus 2
```

#### `podman machine start` / `stop`
*Starts or stops the Podman virtual machine.*
> You must run `start` before you can run any containers. `stop` is a graceful shutdown that preserves the VM's disk state.
```shell
podman machine start
podman machine stop
```

#### `podman machine ls`
*Lists all available Podman virtual machines and their current state.*
```shell
podman machine ls
```

#### `podman machine ssh`
*Connects to the Podman VM with an interactive SSH session.*
> The best way to debug issues within the VM itself or to run commands directly inside it.
```shell
# Open an interactive shell in the default VM
podman machine ssh

# Run a specific command inside the VM without a full shell
podman machine ssh -- rpm -q podman
```

#### `podman machine rm`
*Removes a virtual machine and all its associated files.*
> ☢️ **Destructive Action:** This will delete the VM's disk image. Use with caution.
```shell
# Forcefully remove the VM without a confirmation prompt
podman machine rm --force
```

---
### 📦 Managing Containers
---
*The fundamental commands for the container lifecycle.*

#### `podman run`
*Creates and starts a new container from an image in one step.*
> The most versatile command. Use `-d` for detached (background) mode, `-it` for an interactive terminal, and `--rm` to automatically remove the container when it exits.
```shell
# Run a Postgres database in the background with a volume, env var, and mapped port
podman run -d --name pg -e POSTGRES_PASSWORD=secret -v pgdata:/var/lib/postgresql/data -p 5432:5432 docker.io/library/postgres
```

#### `podman create`
*Creates a new container but does not start it.*
> Useful for setting up a container's configuration so you can inspect or modify it before its first run.
```shell
# Create a container named 'my-app' from the 'fedora' image
podman create --name my-app fedora
```

#### `podman start` / `stop`
*Starts or stops one or more containers.*
> `stop` sends a `SIGTERM` signal first, then `SIGKILL` after a timeout (default 10s).
```shell
podman start my-app
podman stop -t 5 pg
```

#### `podman kill`
*Forcibly stops a container by sending a signal (default `SIGKILL`).*
> This is a harder stop than `podman stop` and should be used when a container is unresponsive.
```shell
podman kill pg
```

#### `podman restart`
*A convenient shortcut to stop and then start a container.*
```shell
podman restart pg
```

#### `podman rm`
*Removes one or more stopped containers.*
> Use `-f` (`--force`) to remove a running container. Use `-v` to also remove any anonymous volumes associated with it.
```shell
# Remove a specific stopped container
podman rm my-app

# Force remove all containers (running and stopped)
podman rm -af
```

#### `podman ps`
*Lists containers. By default, it only shows running containers.*
> The `-a` or `--all` flag is essential for seeing all containers, including stopped ones.
```shell
# List all containers, running and stopped, with their size
podman ps -as
```

#### `podman pause` / `unpause`
*Pauses or unpauses all the processes in a container.*
```shell
podman pause my-app
podman unpause my-app
```

---
### 🔍 Inspecting & Debugging Containers
---
*Look inside your containers to find out what's going on.*

#### `podman logs`
*Fetches the logs of a container.*
> The most important command for debugging. Use `-f` to "follow" the logs in real-time.
```shell
# Follow the logs of the 'pg' container
podman logs -f pg
```

#### `podman exec`
*Executes a new command inside a running container.*
> Your gateway for getting an interactive shell (`-it`) inside a container to debug it from within.
```shell
# Get an interactive bash shell inside the 'pg' container
podman exec -it pg /bin/bash
```

#### `podman inspect`
*Returns detailed, low-level information about an object in JSON format.*
> Use Go templates with `--format` to extract specific fields.
```shell
# Get the container's IP address
podman inspect --format '{{.NetworkSettings.IPAddress}}' pg
```

#### `podman top`
*Displays the running processes inside a container.*
> Like running `ps -ef`, but targeted at a specific container.
```shell
podman top pg
```

#### `podman stats`
*Displays a live stream of resource usage statistics for containers.*
> Shows CPU, memory, network, and block I/O usage in real-time.
```shell
# Stream the resource stats for all running containers, without clearing the screen
podman stats --all --no-stream
```

#### `podman port`
*Lists the port mappings for a container.*
> Quickly shows how a container's internal ports are mapped to your host machine.
```shell
# Show port mappings for the 'pg' container
podman port pg
```

#### `podman diff`
*Inspect changes on a container's filesystem since it was created.*
> `A` = Added, `C` = Changed, `D` = Deleted.
```shell
podman diff pg
```

---
### 🗃️ Managing Pods
---
*Pods are groups of containers that share resources. This is a core concept from Kubernetes, built right into Podman!*

#### `podman pod create`
*Creates a new, empty pod.*
> Pods are often created with port mappings that all containers inside will share.
```shell
# Create a pod for a web application, exposing port 8080 on the host
podman pod create --name web-app -p 8080:80
```

#### `podman run --pod`
*Runs a new container **inside** an existing pod.*
> Containers in a pod can talk to each other over `localhost`.
```shell
# Add a Redis cache and the main app to the 'web-app' pod
podman run -d --pod web-app --name redis-cache docker.io/library/redis
podman run -d --pod web-app --name my-app docker.io/library/my-custom-app
```

#### `podman pod ps`
*Lists all pods.*
> Use `--format` to customize the output.
```shell
# List all pods with their associated container names
podman pod ps --ctr-names
```

#### `podman pod stop` / `rm`
*Stops or removes an entire pod and all of its containers in one command.*
```shell
podman pod stop web-app
podman pod rm web-app
```

---
### 🖼️ Managing Images
---
*Commands for building, distributing, and managing your container images.*

#### `podman build`
*Builds an image from a Containerfile (or Dockerfile).*
> `-t` tags the image. `.` uses the current directory as the build context.
```shell
podman build -t my-custom-app:1.0 .
```

#### `podman images`
*Lists all images stored locally.*
```shell
# Can be aliased as `podman image ls`
podman images
```

#### `podman pull` / `push`
*Downloads an image from a registry or uploads an image to one.*
```shell
podman pull docker.io/library/ubuntu
podman push my-custom-app:1.0 docker.io/myusername/my-custom-app:1.0
```

#### `podman rmi`
*Removes one or more images from local storage.*
```shell
# Remove the ubuntu image
podman rmi docker.io/library/ubuntu

# Remove all images
podman rmi -a
```

#### `podman tag` / `untag`
*Adds or removes a name (tag) from a local image.*
```shell
podman tag my-custom-app:1.0 my-custom-app:latest
podman untag my-custom-app:latest
```

#### `podman save` / `load`
*Saves an image to a tar archive or loads an image from one.*
> The primary way to move images between systems without a registry.
```shell
# Save an image to a tarball
podman save -o my-app.tar my-custom-app:1.0

# Load an image from a tarball
podman load -i my-app.tar
```

#### `podman history`
*Shows the history of how an image was created, layer by layer.*
```shell
podman history my-custom-app:1.0
```

---
### 🤝 Interacting with Kubernetes & Systemd
---
*Seamlessly move your workloads between Podman, Kubernetes, and Systemd.*

#### `podman generate kube`
*Generates Kubernetes YAML from a Podman pod or container.*
> Build and test your app locally in Podman, then generate the Kubernetes manifest to deploy it to a cluster.
```shell
# Create a Kubernetes-compatible YAML file from the 'web-app' pod
podman generate kube web-app > web-app.yaml
```

#### `podman play kube`
*Creates and runs a pod based on a Kubernetes YAML file.*
> Allows you to test Kubernetes manifests locally using Podman before deploying them to a real cluster.
```shell
# Deploy a pod locally using a Kubernetes manifest file
podman play kube web-app.yaml
```

#### `podman generate systemd`
*[DEPRECATED]* Generates systemd unit files for a container or pod.*
> The modern and recommended approach is to use Quadlet files.
```shell
# Generate systemd files for a container named 'my-app'
podman generate systemd --new --files --name my-app
```

#### `podman quadlet`
*Manages declarative container definitions using `.container`, `.pod`, etc., files.*
> Quadlet is the modern, preferred way to run containers as systemd services.
```shell
# List all installed quadlets
podman quadlet list
```

---
### 🌐 Managing Networks
---
*Create custom networks to isolate containers and pods.*

#### `podman network create`
*Creates a new container network.*
```shell
# Create a new bridge network with a specific subnet
podman network create --subnet 192.168.99.0/24 my-app-net
```

#### `podman network ls`
*Lists all available networks.*
```shell
podman network ls
```

#### `podman network inspect`
*Displays detailed configuration for one or more networks.*
```shell
podman network inspect my-app-net
```

#### `podman network connect` / `disconnect`
*Connects or disconnects a container from a network.*
```shell
podman network connect my-app-net some-container
podman network disconnect my-app-net some-container
```

#### `podman network rm` / `prune`
*Removes one or more networks, or all unused networks.*
```shell
podman network rm my-app-net
podman network prune
```

---
### 💾 Managing Volumes
---
*Manage persistent data for your containers.*

#### `podman volume create`
*Creates a new named volume.*
```shell
# Create a new volume for database data
podman volume create my-db-data
```

#### `podman volume ls`
*Lists all available volumes.*
```shell
podman volume ls
```

#### `podman volume inspect`
*Displays detailed information about a volume.*
```shell
podman volume inspect my-db-data
```

#### `podman volume rm` / `prune`
*Removes one or more volumes, or all unused volumes.*
> `prune` is safer as it won't remove volumes currently in use.
```shell
podman volume rm my-db-data
podman volume prune
```

---
### 🧹 System & Cleanup
---
*Commands for managing the Podman environment and cleaning up resources.*

#### `podman system df`
*Shows disk usage for images, containers, and volumes.*
```shell
podman system df
```

#### `podman system prune`
*Removes all unused data: stopped containers, unused networks, and dangling images.*
> ☢️ **Use with care.** By default, it will not remove volumes. Add `--volumes` to also prune unused volumes. This command will remove a stopped `minikube` container.
```shell
# Prune everything, including volumes, without a confirmation prompt
podman system prune -af --volumes
```

#### `podman info`
*Displays detailed system-wide information about the Podman installation.*
```shell
podman info
```