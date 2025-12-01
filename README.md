# DODA-practice
DevOps exercises to understand and practice.



# Docker Basics

## What Is Docker?
Docker uses **images** to create **containers**.  
An *image* is a blueprint, and a *container* is a running instance of that blueprint.

---

## Creating Docker Images

To create your own image, you first need a **Dockerfile**.

### Build an image
```bash
docker build /path/to/Dockerfile
```

### Build and tag an image
```bash
docker build -t repositoryName:1.0.0 /path/to/Dockerfile
```

**Explanation:**
- `-t` adds a **name** and **version tag**
- Example: `myapp:1.0.0`

---

## Listing Images

List all images on your system:

```bash
docker images
```

---

## Running Containers

Run a container from an image:

```bash
docker run imageNameOrId
```

### Useful `docker run` flags

- `-d` - run in background (detached mode)  
- `--rm` - automatically remove container after it stops  
- `-p 8080:80` - map port 8080 on your machine to port 80 inside the container  
- `--name SOMENAME` - give the container a custom name  

**Example:**

```bash
docker run -d --name webapp -p 8080:80 myimage:1.0.0
```

---

## Docker Compose

Docker Compose allows you to run **multiple containers together** that can communicate with each other.  
It requires a `docker-compose.yml` file.

### Start all services
```bash
docker-compose up -d
```

### Stop and remove all services
```bash
docker-compose down
```

# Vagrant & Ansible Basics

## What Is Vagrant?
Vagrant is a tool to **manage virtual machines**.  
A *Vagrant box* is a base image (like Docker image), and a *VM* is a running instance of that box.  
Vagrant uses a **provider** like VirtualBox to create the VM.

---

## Installing Tools

### Ansible
Requires Python. Install with:

```bash
pip install ansible
```

Verify installation:

```bash
ansible --version
```

### Vagrant
Install via package manager or installer. Verify:

```bash
vagrant --version
```

### VirtualBox (Provider)
Make sure the binary is executable:

```bash
VirtualBox --help
```

---

## Creating a VM

### Initialize a Vagrant project
Create a folder and initialize Vagrantfile with a Bento Ubuntu box:

```bash
vagrant init bento/ubuntu-24.04
```

Set a specific version in the `Vagrantfile`:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_version = "202510.26.0"
end
```

### Start the VM
```bash
vagrant up
```

The VM will appear in the VirtualBox UI.  

---

## Networking

- **Default NAT Adapter:** Internet access + port forwarding
- **Host-only Adapter:** Allows host machine to access VM directly:

```ruby
config.vm.network "private_network", ip: "192.168.56.10"
```

Apply network changes:

```bash
vagrant reload
```

---

## SSH Access

Connect to VM:

```bash
vagrant ssh
```

Or via port-forwarded SSH:

```bash
ssh -p 2222 vagrant@localhost
```

Set up passwordless login (from host to VM):

```bash
ssh-copy-id vagrant@192.168.56.10
ssh vagrant@192.168.56.10
```

---

## Increasing VM Resources

Edit `Vagrantfile`:

```ruby
config.vm.provider "virtualbox" do |v|
  v.memory = 4096
  v.cpus = 2
end
```

Apply changes:

```bash
vagrant reload
```

---

## Ansible Ad-hoc Commands

### Inventory (`inventory.cfg`)
```ini
[test]
vagrant@192.168.56.10
```

### Configuration (`ansible.cfg`)
```ini
[defaults]
inventory = inventory.cfg
```

### Test Connection
```bash
ansible test -m ping
```

### Run Commands
```bash
ansible test -m command -a "ls / -hal"
```

### Idempotent Modules
Install Nginx:

```bash
ansible test -m apt -a "update_cache=true name=nginx state=present" --become
```

Start service:

```bash
ansible test -m service -a "name=nginx state=started enabled=true"
```

---

## Ansible Playbook

### Example Playbook (`provisioning.yml`)
```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: Install Nginx
      apt: name=nginx state=present update_cache=true
    - name: (Auto-) Start Nginx
      service: name=nginx state=started enabled=true
    - name: Copy image to server
      copy:
        src: pic.jpg
        dest: /var/www/html/
        owner: www-data
        group: www-data
        mode: "0644"
    - name: Replace the Homepage Title
      replace:
        path: /var/www/html/index.nginx-debian.html
        regexp: "<h1>Welcome to nginx!</h1>"
        replace: "<h1>Welcome to the REMLA course!</h1><img src=\"pic.jpg\" />"
```

### Provisioning with Vagrant
Add to `Vagrantfile`:

```ruby
config.vm.provision :ansible do |a|
  a.compatibility_mode = "2.0"
  a.playbook = "provisioning.yml"
end
```

Run provisioning:

```bash
vagrant provision
```

Or run directly with Ansible:

```bash
ansible-playbook -i inventory.cfg provisioning.yml
```

Check the changes by visiting `<VMIP>:80` in your browser.

---

## Managing the VM

- Stop VM:

```bash
vagrant halt
```

- Pause/Resume VM:

```bash
vagrant suspend
vagrant resume
```

- Remove VM:

```bash
vagrant destroy
```

# Kubernetes

## 1. Start the Cluster

To begin working with Kubernetes locally, you will use **Minikube**, a lightweight Kubernetes implementation that runs on your local machine. It provides a single-node cluster suitable for learning and development.

### Start Minikube with the Docker Driver

Using Docker as the backend avoids the overhead of running a full VM.

```bash
minikube start --driver=docker
```

This command:
- Creates a single-node Kubernetes cluster
- Configures `kubectl` automatically to use the `minikube` context
- Pulls required Kubernetes images

### Enable the Ingress Addon

Ingress controllers are not enabled by default. You need one to support HTTP routing into your cluster.

```bash
minikube addons enable ingress
```

This installs the **Ingress-Nginx Controller**, which will manage incoming HTTP(S) traffic.

### Verify the Cluster and Open Dashboard

Check that the control plane is running:

```bash
kubectl cluster-info
```

Run the Minikube dashboard:

```bash
minikube dashboard
```

The dashboard provides:
- Live overview of pods, services, deployments
- Integrated logs view
- Quick access to exec into containers
- Visual inspection of resource configuration

---

## 2. Pods

A **Pod** is the smallest deployable unit in Kubernetes. It typically runs one container but can run more if needed.

### Deploying a Pod

You create a Pod using a YAML manifest such as `simple-pod.yml`, which runs `nginx:1.14.2`.

Deploy it:

```bash
kubectl apply -f simple-pod.yml
```

### Inspecting Pods

List running Pods:

```bash
kubectl get pods -o wide
```

Get detailed configuration and status:

```bash
kubectl describe pods
```

This shows:
- Container image
- Events
- Labels
- IP and node placement
- Volumes
- Environment variables

### Interacting with a Pod

Forward a port from your machine to the Pod:

```bash
kubectl port-forward nginx 1234:80
```

Check logs:

```bash
kubectl logs nginx
```

Enter the Pod interactively:

```bash
kubectl exec -it nginx -- bash
```

### Deleting Pods

```bash
kubectl delete -f simple-pod.yml
```

This stops and removes the Pod from the cluster.

---

## 3. Services and Ingress

Pods are ephemeral and not meant to be accessed directly. **Services** provide stable networking while **Ingress** exposes HTTP(S) traffic externally.

---

### NodePort Service

A NodePort:
- Exposes a Service on a static port between 30000-32767
- Allows external access via `<NodeIP>:<NodePort>`

After defining a NodePort in `service.yml`, apply it:

```bash
kubectl apply -f service.yml
```

Check services:

```bash
kubectl get services
```

Find the node IP:

```bash
kubectl get nodes -o wide
```

Minikube simplifies access:

```bash
minikube service my-service --url
```

This automatically opens a tunnel if required.

---

### Ingress

Ingress provides:
- Host-based routing (e.g., api.example.com)
- Path-based routing (e.g., /api, /dashboard)
- Centralized entry point on port 80/443

After defining an Ingress referencing a ClusterIP service:

```bash
kubectl apply -f ingress.yml
```

Start the tunnel to expose ports 80/443:

```bash
minikube tunnel
```

Then open:

- http://localhost

---

## 4. Deployments

A **Deployment** provides:
- Replica management
- Rolling updates
- Self-healing
- Declarative Pod templates

### Apply the Deployment

```bash
kubectl apply -f deployment.yml
minikube tunnel
```

List Pods:

```bash
kubectl get pods
```

### Self-Healing Capabilities

Delete a Pod:

```bash
kubectl delete pod <pod-name>
```

Kubernetes will recreate the missing Pod automatically to match the desired replica count.

### Scaling a Deployment

Increase replicas in the YAML:

```yaml
replicas: 5
```

Apply changes:

```bash
kubectl apply -f deployment.yml
```

Kubernetes will add or remove Pods to match the new number.

---

## 5. Environment Variables and Storage

### ConfigMap and Secret

**ConfigMaps** store configuration that is not sensitive.

**Secrets** store sensitive data encoded in base64.

Apply both resources:

```bash
kubectl apply -f environment.yml
```

View them in the Minikube dashboard to inspect stored keys and values.

---

### Pod Environment Injection

A Pod can load values from ConfigMaps and Secrets using `env.valueFrom`.

After deploying:

```bash
kubectl port-forward showenv 1234:8080
```

Open the app to verify environment variables:
- `USERNAME` (from ConfigMap)
- `FOO` (from Secret)

---

### Volumes

A **hostPath** volume maps a directory from the Kubernetes node into a container.

Since the volume config cannot be merged dynamically, delete and re-apply:

```bash
kubectl delete -f environment.yml
kubectl apply -f environment.yml
```

### Test Persistence

Write a file:

```bash
kubectl exec -it showenv -- touch /data_inside/i_was_here
```

Restart Pod and verify:

```bash
kubectl exec -it showenv -- ls /data_inside
```

### Inspect Host Path

Minikube runs inside a VM or container. To inspect the actual path:

```bash
minikube ssh
ls /data_outside
```

This confirms that the volume persists across Pod recreation.




