# Docker ignore file

File: .dockerignore 

Indicates files and folders to not get copied in the COPY instruction

---


# ARGuments & ENVironment Variables

## Environmet variables
In Dockerfile, application code and --env on docker run command

```
(...)
# ENV {NAME} {DEFAULT_VALUE}
ENV PORT 80
(...)
```
This way the {NAME} environment variable can be consumed from inside the container. The ENV keys can be exposed.

```
(...)
ENV PORT 80
EXPOSE $PORT
(...)
```

Assign on container creation:

> docker run --env PORT=4200 (...)


# .env file
```
(...)
PORT=4200
(...)
```
instead of using --env argument on run we can reffer to a environment file
> docker run --env-file .env (...)

# Arguments
Inspected on image build-time. Dockerfile:

```
(...)
# ARG ARG_NAME=ARG_DEFAULT_VALUE
ARG DEFAULT_PORT=80

ENV PORT $DEFAULT_PORT

EXPOSE $PORT
(...)
```

> docker build --build-arg DEFAULT_PORT=4200

---

# Networks

- Connect container to the WWW
- Connect container to the host machine
- Connect container to another container


## Connect container to the WWW
It's enabled by default

## Connect container to the host machine
URL destination should be **host.docker.internal**, not *localhost*.

## Connect container to another container
The dirty way: We can simply point to the container internal IP
> docker inspect CONTAINER_ID/CONTAINER_NAME
```
(...)
    "Config": {
        (...)
        "NetworkSettings": {
            (...)
            "IPAddress": "172.17.0.2",
            (...)
        },
        (...)
    },
(...)
```

The clean Way: Create a Container Network and add both containers to this network
> docker run --network my_network (...)

**Networks wont be created by docker automatically (unlike Volumes)!**

> docker network create myNet

> docker network ls
```
NETWORK ID     NAME      DRIVER    SCOPE
1557fb5c7f7d   bridge    bridge    local
e1ca5d33b57f   host      host      local
f9ff0510dc69   myNet     bridge    local
f0fd11b72e08   none      null      local
```

Now  as both containers are inside the same network they can find each other by hostname (container name is hostname)

---

# Docker Compose

To replace multiple docker build / run in one config file and orchestrate multiple containers (or just one)

## docker-compose.yml / .yaml
```
# docker-compose specification version
version: "3.8"

# services will be the containers
services: 
  mongodb:
    image: "mongo"
    # container_name: "mongodb" generic name will be assigned, and it's the best way to use it
    volumes:
      - data:/data/db
    # environment:
      # MONGO_INITDB_ROOT_USERNAME: "max"
      # MONGO_INITDB_ROOT_PASSWORD: "secret"
      # - MONGO_INITDB_ROOT_USERNAME=max works the same way, this works as a single value, above are key - value
    env_file:
      - ./env/mongo.env
    # networks:
    #   - myNet
  backend:
    # image: "goals-node"
    # docker can use a Dockerfile to create an image
    build: ./backend
    # Longer alternative with extra args
    # build: 
    #   context: ./backend
    #   dockerfile: Dockerfile # Optional if it's called "Dockerfile"
    #   args:
    #     some-arg:1
    ports:
      - "80:80"
    volumes:
      # named volume
      - logs:/app/logs
      # Bind mount in relative path
      - ./backend:/app
      # anonymous volume
      - /app/node_modules
    # environment:
    #   MONGO_INITDB_ROOT_USERNAME: max
    #   MONGO_INITDB_ROOT_PASSWORD: secret
    env_file:
      - ./env/backend.env
    depends_on:
      # only when these started ok, "backend" will be started
      - mongodb
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      # code changes are taken into consideration
      - ./frontend/src:/app/src
    # -it equivalient part 1 
    stdin_open: true
    # -it equivalient part 2
    tty: true
    depends_on:
      - backend

volumes:
  data:
  logs:

# By default containers will be removed on stop
# By default these containers will belong to the same network
```

> docker-compose up

Will start everything (optional -d for detatch) (optional --build builds the images)

> docker-compose down

Stops containers if running and removes them. With -v will also remove volumes

# "Utility" Containers
Containers holding only environment and no app to (perhaps) make an app.

## Running commands in containers
Start the container in interactive mode and dettatched
> docker run -it -d **CONTANER**

Send command
> docker exec -it **CONTANER** *command arg1 arg2 ...*

### Ex, creating a node project
```
> docker run -it -d --name myNodeContainer node 
> docker exec -it myNodeContainer npm init
```

Start container overriding default image command
> docker run -it node npm init

### Ex: Use an utility container to create a node project in the host machine
Dockerfile:
```
FROM node:14-alpine
WORKDIR /app
```
> docker build -t node-util .

> docker run -it -v $PWD:/app node-util npm init

## Dockerfile ENTRYPOINT
It's a non overrideable and pre-appended to all the arguments send to the docker exec command.

Ex node npm Dockerfile
```
FROM node:14-alpine
WORKDIR /app
ENTRYPOINT ["npm"]
```
> docker build -t mynpm .

> docker run -it -v $PWD:/app mynpm init

## App argumments docker-compose
This way it starts only one service of the docker-compose.yaml
> docker-compose run **SERVICE_NAME** *arg1 arg2 ...*

Beware: **SERVICE_NAME**  wont be removed when launched like this

## Autoremove in docker-compose when running single service
> docker-compose run --rm **SERVICE_NAME** *arg1 arg2 ...*

