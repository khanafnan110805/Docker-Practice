# Docker Notes (Personal Understanding)

## Core Mental Model

Docker has 2 main things:

### 1. Image

Blueprint / template.

Like:

- Node environment
- MongoDB setup
- Ubuntu setup

Image is NOT running.

Example:

```bash
docker pull mongo
```

This downloads the Mongo IMAGE.

That’s why after pulling:

```bash
docker images
```

shows:

```txt
mongo
```

So:

- `mongo` = image
- NOT container

---

### 2. Container

Running instance created FROM image.

Example:

```bash
docker run mongo
```

This:

- creates container FROM mongo image
- starts container

So:

```bash
docker ps
```

shows RUNNING containers.

NOT images.

---

# Important Difference

## docker images

Shows:

- downloaded images
- built images

Example:

```txt
mongo
node
my-app:1.0
```

These are templates/blueprints.

---

## docker ps

Shows:

- RUNNING containers only

Example:

```txt
mongodb
mongo-express
my-app
```

These are active runtime instances.

---

## docker ps -a

Shows:

- running containers
- stopped containers

Very important.

Because stopping container does NOT delete it.

---

# Pulling Images

Download image from Docker Hub:

```bash
docker pull mongo
```

```bash
docker pull mongo-express
```

```bash
docker pull node
```

This only downloads images locally.

Nothing runs yet.

---

# Running Containers

Create + start container from image:

```bash
docker run mongo
```

or:

```bash
docker run mongo-express
```

But basic docker run uses defaults only.

Real projects need:

- ports
- passwords
- usernames
- networking

That’s why long commands exist.

Example:

```bash
docker run -d \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=password \
--name mongodb \
mongo
```

---

# Why Docker Compose Exists

Instead of writing huge docker run commands repeatedly,
we create a YAML file.

Example:

```yaml
services:
  mongodb:
    image: mongo

  mongo-express:
    image: mongo-express
```

Then simply run:

```bash
docker compose -f mongo.yaml up -d
```

This automatically:

- creates containers
- creates network
- applies env variables
- starts services together

Compose basically manages multiple connected containers as one application stack.

---

# Building My Own App Image

Dockerfile creates custom image for my app.

Build image:

```bash
docker build -t my-app:1.0 .
```

This:

- reads Dockerfile
- packages app
- creates image

IMPORTANT:
This does NOT run container.

---

# Run My App Container

```bash
docker run -d -p 3000:3000 my-app:1.0
```

This creates + starts container from image.

---

# Why -p Exists

```bash
-p 3000:3000
```

means:

```txt
host-port : container-port
```

So:

- localhost:3000 on my PC
  connects to:
- port 3000 inside container

---

# Enter Inside Running Container

```bash
docker exec -it <container-id> /bin/sh
```

Example:

```bash
docker exec -it f7b6bb52c5a9 /bin/sh
```

Meaning:

- docker exec
  = run command inside existing container

- -it
  = interactive terminal mode

- /bin/sh
  = lightweight Linux shell

Used for:

- debugging
- checking files
- running Linux commands
- inspecting container manually

---

# Why /bin/bash Failed Earlier

Alpine Linux images are minimal.

They usually contain:

```txt
/bin/sh
```

NOT:

```txt
/bin/bash
```

---

# Stopping Containers

```bash
docker stop <container-id>
```

Container stops BUT still exists.

After stopping:

- docker ps → won't show it
- docker ps -a → WILL show it

---

# Removing Containers

```bash
docker rm <container-id>
```

Deletes stopped container completely.

If container is still running:

```txt
You cannot remove a running container
```

So:

- stop first
- then remove

Shortcut:

```bash
docker rm -f <container-id>
```

Force stop + remove together.

---

# Removing Images

```bash
docker rmi <image-id>
```

Deletes image.

But image cannot be deleted if:

- some container still references it
- even stopped container

That’s why sometimes:

- remove container first
- then remove image

---

# Real Workflow After Code Changes

If I change:

- code
- Dockerfile
- file structure

Then:

## 1. Stop old container

```bash
docker stop <container-id>
```

## 2. Remove old container

```bash
docker rm <container-id>
```

## 3. Rebuild image

```bash
docker build -t my-app:1.0 .
```

## 4. Run new container

```bash
docker run -d -p 3000:3000 my-app:1.0
```

---

# Biggest Docker Understanding

Docker is basically isolated environments.

Main benefit:
different projects can use:

- different Node versions
- different dependencies
- different runtimes

WITHOUT messing up my actual PC setup.

Example:

- Project A → Node 20
- Project B → Node 14

Both can run together using separate containers.

That’s the actual power of Docker.
