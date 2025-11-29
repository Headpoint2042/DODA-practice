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




