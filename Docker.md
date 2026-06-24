
Docker runs apps as **isolated containers** built from **images**, defined via **Dockerfile**, and managed via **Docker Compose** for multi-service apps.

## 1. Mental flow (must know)

```
Dockerfile → Image → Container → Running App
           (build)   (run)
```

## 2. Context

Controls _which Docker engine you are using_.

```bash
docker context ls
docker context show
docker context use desktop-linux   # or default
```

If you mess this up → containers “disappear”.

## 3. Daily essential commands

### Check status

```bash
docker ps          # running containers
docker ps -a       # all containers
docker images      # stored images
```

---

### Run / stop containers

```bash
docker run IMAGE                  # start container
docker run -d IMAGE               # background
docker run -it IMAGE bash         # shell access

docker stop CONTAINER
docker start CONTAINER
docker restart CONTAINER
docker rm CONTAINER
```

### Debug

```bash
docker logs CONTAINER
docker exec -it CONTAINER bash
docker inspect CONTAINER
```

## 4. Docker Compose 

Used for full stacks (API + DB + Redis + workers)

```bash
docker compose up -d      # start everything
docker compose down       # stop everything
docker compose ps         # check services
docker compose logs       # see logs
docker compose build      # rebuild images
```

This will build the container from the image then start it
```bash
docker compose up <name of container> --build
```
---

## 5. Images (build system)

```bash
docker build -t name .    # build image
docker pull image         # download image
docker rmi image          # delete image
```

## 6. System cleanup (optional but useful)

```bash
docker system df
docker system prune
docker image prune
```

---

## 7. Volumes (data persistence)

```bash
docker volume ls
docker volume create NAME
docker volume rm NAME
```

## 8. Networks (containers talking)

```bash
docker network ls
docker network inspect NAME
```

## 9. Your real debugging checklist (use this always)

If something “doesn’t work”:

```bash
docker context show
docker ps -a
docker compose ps
docker logs <container>
```
