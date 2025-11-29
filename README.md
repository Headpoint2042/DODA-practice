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



