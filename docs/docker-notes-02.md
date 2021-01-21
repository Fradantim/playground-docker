- - - - - - - - - - - - - - - - - -

## Docker run interactive
docker run -it IMAGE_ID

## Docker start interactive
docker start -i CONTAINER_ID/CONTAINER_NAME

- - - - - - - - - - - - - - - - - -

> docker ps -a
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                      PORTS     NAMES
c12019059022   ca586a852370   "python rng.py"          3 minutes ago   Exited (0) 2 minutes ago              sad_taussig
eae76cba6771   b53cc11d746e   "docker-entrypoint.s…"   46 hours ago    Exited (137) 46 hours ago             zealous_snyder
86c1f062e485   hello-world    "/hello"                 2 days ago      Exited (0) 2 days ago                 agitated_mahavira
```

## Delete container
> docker rm CONTAINER_ID/CONTAINER_NAME (...) (...)

> docker images
```
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
<none>        <none>    ca586a852370   9 minutes ago   885MB
node          14        72aaced1868f   5 days ago      942MB
python        latest    d1eef6fb8dbe   5 days ago      885MB
hello-world   latest    bf756fb1ae65   11 months ago   13.3kB
```

## Delete image
> docker rmi IMAGE_ID (...) (...)

## Delete all unused images
> docker image prune

- - - - - - - - - - - - - - - - - -

## Remove stopped container automatically
> docker run --rm IMAGE_ID

- - - - - - - - - - - - - - - - - -

## Inspect image
> docker image inspect 72aaced1868f
```
[
    {
        "Id": "sha256:72aaced1868fe290b28e2ef17dbfbecbcb5f026a939a4669f5efe07c94cb90ef",
        "RepoTags": [ "node:14" ],
        "RepoDigests": [ "node@sha256:75e1dc0763f97d0907b81e378d0242ab9034fb54544430898b99a3ac71fa0928" ],
        "Parent": "",
        (...)
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:0279b13c6fc69327776f36c3e76faafb3d15dca370a91eefdd22641b2c50f257",
                "sha256:de1bb73bcb8b1510a25b52bed61cd13ec262df39e02b0cbdf9830bddd7edb8a8",
                "sha256:0a1c0b220d6e002ff35117e8badacac0548e974a0eaf1fc37a9a592aba9ab930",
                "sha256:8f82110acbe47306a09cdc1c4c23d286f50b8eb0de69b1ed4a1c0905f59c7f40",
                "sha256:75298d0cbb21ff5da354e37dcc6638896863e76340683a80ffb3fadf3cbce67b",
                "sha256:5a5c0a2eda051d78c47658ae2efc29f7f4549eb4ca3f34316105c40345ffa873",
                "sha256:9499c8f596a0c905497768a8bca5c04a0f3f86cb2722646ee2912281a3ad185c",
                "sha256:f92bc4578fac8c57a438824bdaae3ca757eeaa87e879b020965564b2f5c59986",
                "sha256:884645749ee7f943593083aaa7301d24ea6007497852fce88dccc2e6a886a461"
            ]
        },
        "Metadata": { "LastTagTime": "0001-01-01T00:00:00Z" }
    }
]
```

- - - - - - - - - - - - - - - - - -

## copy file from outside the container
> docker cp (file or directory ex 'file.dat' or 'dummy/.') (CONTAINER_ID/CONTAINER_NAME):(path ex '/test')

- - - - - - - - - - - - - - - - - -

## Tag image
> docker build -t goals:(TAG)

ex:

> docker build -t goals:1.3
> docker build -t goals:latest

## Re-Tag

> docker tag IMAGE_ID:old-tag IMAGE_ID:new-tag 

## give container a name
> docker run --name TheName IMAGE_ID

- - - - - - - - - - - - - - - - - -

## Dockerize BOTH apps - the Python and the Node app.

### node

```
1) Create appropriate images for both apps (two separate images!). HINT: Have a brief look at the app code to configure your images correctly!
FROM node

WORKDIR /app

# cacheable
COPY package.json /app

# cacheable
RUN npm install

COPY . /app

EXPOSE 3000

CMD [ "node", "server.js" ]
- - - -

> docker build .
(...)
Successfully built ae588b8022ae
```
```
2) Launch a container for each created image, making sure, that the app inside the container works correctly and is usable.
> docker run -d -p 3000:3000 ae588b8022ae
498f33fc61d807d3ceb556f6ad2b9b2c111035f807e8bbbb29ec0497e6069a2b

> docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                    NAMES
498f33fc61d8   ae588b8022ae   "docker-entrypoint.s…"   20 seconds ago   Up 19 seconds   0.0.0.0:3000->3000/tcp   pensive_villani

>  docker stop 498f33fc61d8
498f33fc61d8
```
```
3) Re-create both containers and assign names to both containers. Use these names to stop and restart both containers.
> docker run -d -p 3000:3000 --name TheNodeApp ae588b8022ae
a020a5bfb144210e24104feb1b2e48936f3192acd5e9fb5bb0d754b050cff6dc

> docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                    NAMES
a020a5bfb144   ae588b8022ae   "docker-entrypoint.s…"   4 seconds ago   Up 4 seconds   0.0.0.0:3000->3000/tcp   TheNodeApp

> docker stop TheNodeApp
TheNodeApp

> docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


> docker start TheNodeApp
TheNodeApp

> docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS         PORTS                    NAMES
a020a5bfb144   ae588b8022ae   "docker-entrypoint.s…"   About a minute ago   Up 5 seconds   0.0.0.0:3000->3000/tcp   TheNodeApp
```
```
4) Clean up (remove) all stopped (and running) containers, clean up all created images.
> docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED             STATUS                            PORTS     NAMES
a020a5bfb144   ae588b8022ae   "docker-entrypoint.s…"   2 minutes ago       Exited (137) About a minute ago             TheNodeApp
498f33fc61d8   ae588b8022ae   "docker-entrypoint.s…"   5 minutes ago       Exited (137) 4 minutes ago                  pensive_villani
ffe96ddab8f8   ca586a852370   "python rng.py"          54 minutes ago      Exited (0) 54 minutes ago                   clever_lewin
c959b2568f96   ca586a852370   "python rng.py"          About an hour ago   Exited (0) About an hour ago                priceless_ride
2354154305e2   ca586a852370   "python rng.py"          About an hour ago   Exited (1) About an hour ago                amazing_cohen
15d2f8aac807   2ca6c2f824a0   "docker-entrypoint.s…"   47 hours ago        Exited (137) 47 hours ago                   affectionate_haibt
2184b80056d8   2ca6c2f824a0   "docker-entrypoint.s…"   47 hours ago        Exited (137) 47 hours ago                   heuristic_einstein
9597dfc8df1a   node           "docker-entrypoint.s…"   2 days ago          Exited (0) 2 days ago                       musing_hodgkin
d023ffe017e1   node           "docker-entrypoint.s…"   2 days ago          Exited (0) 2 days ago                       stoic_cray
b5b664631da9   7505e359b55b   "docker-entrypoint.s…"   2 days ago          Exited (137) 2 days ago                     crazy_hofstadter
c0748a70daa2   hello-world    "/hello"                 2 days ago          Exited (0) 2 days ago                       dazzling_merkle
86c1f062e485   hello-world    "/hello"                 2 days ago          Exited (0) 2 days ago                       agitated_mahavira

> docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
a020a5bfb144210e24104feb1b2e48936f3192acd5e9fb5bb0d754b050cff6dc
498f33fc61d807d3ceb556f6ad2b9b2c111035f807e8bbbb29ec0497e6069a2b
ffe96ddab8f830730b4a37ca930a49b9ebc649bb061ba665beb2def32cbf15c4
c959b2568f9614ea648a429299c6ba21583eb75c614769ebe16fdf97bb5a6df3
2354154305e25bcc3bf644b5341aa868d2de06ecec7677899caa8034a6e67943
15d2f8aac807761e23abd03ece4062e197c1ba2bf6ed7bf57f3011326a355bf2
2184b80056d8e0b4986efa19b7927baed32e2ac538e5f075694cf9988f6ea24e
9597dfc8df1aae8076149afa613bb1d7728c70ae0765ae4ec8e027737afcf306
d023ffe017e115425e78f8299d08487283379f5a671a6021ab6ca20e90efa738
b5b664631da98ed3a5a475bdcd52b2de18185455d28a67d0f7124c979cb085ba
c0748a70daa2a1853df6b46619e8cb48c41e6cf33a4f65aaf665d98b1dfc0261
86c1f062e485fa70b638e66275b15e9886adba4be5a32dbd4a28774b116887aa

Total reclaimed space: 585.3kB

> docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
```
5) Re-build the images - this time with names and tags assigned to them.
> docker build . -t the-node-app-image:1.0
(...)
Successfully built ae588b8022ae
Successfully tagged the-node-app-image:1.0
```
```
6) Run new containers based on the re-built images, ensuring that the containers are removed automatically when stopped.

> docker run --rm -d -p 3000:3000 the-node-app-image:1.0
af563da278ed64b935a2e273e59a391b11cb29c73ec34e4de96dd332e40a4eca

> docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                    NAMES
af563da278ed   the-node-app-image:1.0   "docker-entrypoint.s…"   26 seconds ago   Up 25 seconds   0.0.0.0:3000->3000/tcp   pedantic_mcclintock

> docker stop pedantic_mcclintock
pedantic_mcclintock

>  docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

---

## Add local repository

(just created the fradantim/node-hello-world docker hub repository)

> docker build -t fradantim/node-hello-world .
```
Sending build context to Docker daemon   16.9kB
(...)
Successfully built d66a3fe2852d
Successfully tagged fradantim/node-hello-world:latest
```

## Docker login to remote repository

> docker login
```
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: fradantim
Password: 
WARNING! Your password will be stored unencrypted in /home/fradantim/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

## Push to remote repository

> docker push fradantim/node-hello-world
```
Using default tag: latest
The push refers to repository [docker.io/fradantim/node-hello-world]
aa4af40637b2: Pushed 
126b51f4899a: Pushed 
e17b2aabc7b7: Pushed 
353ea3d9525e: Pushed 
884645749ee7: Mounted from library/node 
f92bc4578fac: Mounted from library/node 
9499c8f596a0: Mounted from library/node 
5a5c0a2eda05: Mounted from library/node 
75298d0cbb21: Mounted from library/node 
8f82110acbe4: Mounted from library/node 
0a1c0b220d6e: Mounted from library/node 
de1bb73bcb8b: Mounted from library/node 
0279b13c6fc6: Mounted from library/node 
latest: digest: sha256:1b137413661d05453fccd8f13a2a9359acffc7a23861824b064cc3479c14c70e size: 3048
```

## Pull from remote repository

> docker pull fradantim/node-hello-world
```
Using default tag: latest
latest: Pulling from fradantim/node-hello-world
Digest: sha256:1b137413661d05453fccd8f13a2a9359acffc7a23861824b064cc3479c14c70e
Status: Image is up to date for fradantim/node-hello-world:latest
docker.io/fradantim/node-hello-world:latest
```

---

# Volumes

With volumes (i.e.) /app/feedback can be mounted in the host fs, the app will see this directory as in another device.

## Anonymous Volume (Volatile)
Anonymous Volume -> gets deleted on container removal

Dockerfile:

```
(...)
VOLUME ["/app/feedback"]
(...)
```
or
> docker run **-v /app/feedback** feedback-node:volumes

## Named Volume (Non volatile )

Create a volume, since it's named it will survive after container removal
> docker run **-v feedback:/app/feedback** feedback-node:volumes

List Volumes
> docker volume ls
```
DRIVER    VOLUME NAME
local     feedback
```

Delete unused Volumes
> docker volume rm VOLUME_NAME

> docker volume prune

## Read-only Volumes
> docker run -v /home/fradantim/feedback-app/app:/app:**ro** feedback-node:volumes 

By default these are rw

## Priorities
Always keep the options from the most specific config.
> docker run -v /app/feedback:ro -v /home/fradantim/feedback-app/app:/app feedback-node:volumes 

- /app is binded, persistent and read-write, except for
- /app/feedback is anonymous (volatile) and read only

## Bind Mounts

Volumes are managed by docker, and sometimes we dont know where it locates it. With Binds we are completely aware of where those went. Useful for persistent and **editable** data. A bind is not a Volume nor managed by Docker.

> docker run -v feedback:/app/feedback **-v /home/fradantim/feedback-app/app:/app** feedback-node:volumes 
```
Here we are binding the /app container directory to the /home/fradantim/feedback-app/app host directory. This is an absolute directory.
```