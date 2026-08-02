When using a System there are 4 layer:

```
[APPLICAIONS] 
[OS Application] <- Docker virtualizes this  | VM Virtualizes
[OS Kernel]                                  | Both
[System Hardware]
```

```
Dockerfile → Image → Container → Running App
           (build)   (run)
```

**Writting a Dockerfile**
a base image that have the core softare that we need is installed, like Nodejs or python. they are literally  a linux distro but with the a required software.

`FROM`

Dockerfile can be build using
`docker build -t <name> <path of the dockerfile>`

Docker runs apps as **isolated containers** built from **images**, defined via **Dockerfile**, and managed via **Docker Compose** for multi-service apps.

**HealthCheck in docker**

they are used to know if the applicaiton that is hosted in the container actually work or just contianer is running. we can either check the health using terminal, Dockerfile or compose yml

```
# Before staring our service
HEALTHCHECK --interval=30s --timeout=5s --retries=3 CMD curl --fail http://localhost:8000 || exit 1
```

**Docker compose**

```yml
services:
	<container_name>:
	    build:
	      context: ./Backend
	      dockerfile: Dockerfile
		image: <image name>
		ports:
		  - "2020:2020"
		environments:
			ENV1=sec
        healthcheck:
           test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
           interval: 30s
           timeout: 5s
           retries: 3
           start_period: 10s    
		depends_on:
			- <name of another container on which it depnds on>
	
```

health check of the above service is done we can use its health with another service in the depends.   

<br />

i might need to to use variables instead of hard-codded thing in say secrets in that case we use

```
environments:
	ENV1=${SECRET_}
```

**2. Context  and engine**

Controls _which Docker engine you are using_.

```bash
docker context ls
docker context show
docker context use desktop-linux   # or default
```

If you mess this up → containers “disappear”.

**Daily essential commands**

Check status

```bash
docker ps          # running containers
docker ps -a       # all containers
docker images      # stored images
```

Run / stop containers

```bash
docker run IMAGE                  # start container
docker run -d IMAGE               # background
docker run -it IMAGE bash         # shell access

docker stop CONTAINER
docker start CONTAINER
docker restart CONTAINER
docker rm CONTAINER
```

Debug

```bash
docker logs CONTAINER
docker exec -it CONTAINER bash
docker inspect CONTAINER
```

**Docker Compose**

Used for full stacks (API + DB + Redis + workers)

```bash
docker compose up -d      # start everything
docker compose down       # stop everything
docker compose ps         # check services
docker compose logs       # see logs
docker compose stop       # stop container without deleting it
docker compose build      # rebuild images
```

This will build the container from the image then start it

```bash
docker compose -f <docker compose file path> up <name of container> --build
```

**Images (build system)**

```bash
docker build -t name .    # build image
docker pull image         # download image
docker rmi image          # delete image
```

**System cleanup (optional but useful)**

```bash
docker system df
docker system prune
docker image prune
```

**Volumes (data persistence)**

```bash
docker volume ls
docker volume create NAME
docker volume rm NAME
```

**Networks (containers talking)**

```bash
docker network ls
docker network inspect NAME
```

**debugging checklist**

If something “doesn’t work”:

```bash
docker context show
docker ps -a
docker compose ps
docker logs <container>
```

<br />

**Layer Caching**

It is highly recommended to copy `requirements.txt` first **to take full advantage of Docker's build caching mechanism, which drastically speeds up container build times**. as docker checks sequentially where things have changed and the starts rebulding it from there if we copy all files from all together and then install the requreiments then all the lib will downloaded even if it was not changed so we copy things that which will not change often above the thing that will be frequently changed

<br />

**Docker Multistage build**

This is a techinique in which we make 2 image. one for building and another for running. this is used to reduce the image size by multiple times. say we have a cpp code what is needed to be compiled and then run. we dont need all the things that is needed during compiling and code when running the compiled binaries. so we make a image that will compile and then we will copy that complied binary in another where we will use smaller base image (scrach or alpine) and then also not install things like gcc or other compliler as we dont needed it while running it. this will reduce the size of the docker image.

eg

```dockerfile
# 1) Build wheels
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --upgrade pip 
 && pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

# 2) Runtime
FROM python:3.11-slim
WORKDIR /app

# install from prebuilt wheels (no compilers here)
COPY --from=builder /wheels /wheels
RUN pip install --no-cache-dir /wheels/*

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

