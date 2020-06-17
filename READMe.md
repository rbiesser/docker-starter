# Docker Starter
Follow the getting started tutorial from https://github.com/docker/getting-started as an introduction or keep reading to refresh your memory on common commands.

## 1. Install Docker Desktop
https://docs.docker.com/get-docker/

Docker is cross platform. You will find an installer for all popular operating systems. What that means is that you can develop your application with Docker on one machine and know that it will run as you intended it on any other machine. Any software dependencies will be maintained within your code repository.


You can familiarize yourself with the Docker architecture by reading the Docker [Get Started Overview](https://docs.docker.com/get-started/overview/). What you need to know is that Docker installs a command line utility to manage an isolated workspace. On Mac and Windows, Docker creates a Linux virtual machine to host containers while Linux users can take advantage of features already built into Linux. There is also an option to choose Windows containers if you are on a Window host, but let's assume we are working with Linux containers.

### Access the MobyLinux VM’s file system

```bash
# Run this from your regular terminal on Windows / MacOS:
docker container run --rm -it -v /:/host alpine

# Once you're in the container that we just ran, run this:
chroot /host
```

https://nickjanetakis.com/blog/docker-tip-70-gain-access-to-the-mobylinux-vm-on-windows-or-macos

Now you can browse to the `/var/lib/docker/` directory to see all of the docker files. On a Linux host, you can just browse directly to this directory directly without needing to connect to the docker host first. You will most likely not need to login to the docker host vm as most information can be obtained through the CLI, but this demonstrates where that information lives.

### Docker CLI
Get familiar with the command line interface. Docker also ships with the Docker Dashboard but the CLI can be more expressive. Run docker without any arguments or with the `--help` flag to see information about commands you can perform with docker. In fact, add the `--help` flag to any docker command to get more information.

```bash
docker --help
```

At first view the docker architecture and the accompanied CLI can appear intimidating with all of its capabilities, but the official documentation breaks it down easy to get you started. Part of what makes it confusing is that there are many aliases to achieve the same function, so find the one you are most comfortable with but remember the concepts.

Many of the CLI commands are based on Linux counterparts which makes them seem more natural. Look for these options in any management command.s

```bash
COMMAND
ls        # List
rm        # Remove
inspect   # Display detailed information
prune     # Remove all unused
```

## 2. Pick an Image
https://hub.docker.com/

A Docker image is the set of instructions that will be used to create a container. DockerHub hosts many official images of popular applications that you can use as as starting point for your project, and can also host your own images.

### Docker maintains a local repository of images.

```bash
docker images   # List images alias

###

docker image COMMAND
```

### Starting a Container

```bash
docker run -d -p 80:80 docker/getting-started
```
- `-d` - run the container in detached mode (in the background)
- `-p 80:80` - map port 80 of the host to port 80 in the container
- `docker/getting-started` - the image to use

The image is added to your local repository and the container is started. If you do not specify a name for the container it will given a random name. 

### Build your own

https://docs.docker.com/engine/reference/builder/

#### Create a file name `Dockerfile`
```
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "/app/src/index.js"]
```

#### Build the container

```bash
docker build
```

Every command in the Dockerfile is saved in the image history and every change to the file system creates a new layer on the host vm.

## 3. Working with Containers

**In general, each container should do one thing and do it well.**

### Containers exist with a running state

```bash
docker container COMMAND
###

docker ps   # List containers alias
docker stop <the-container-id>    # stop a running container
docker rm <the-container-id>      # Remove a stopped container
```
- A container must be stopped before you can remove it or use the `-f` flag, for force, to perform in one step.
- Each container also gets its own "scratch space" to create/update/remove files.
- Each container starts from the image definition each time it starts.

## 4. Containerize your Application

### Volumes allow persistent data

```bash
docker volume COMMAND
```

#### Create a named volume

```bash
docker volume create todo-db
```

#### Start a container with a mounted volume

```bash
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```
- `-v todo-db:/etc/todos` tells docker to mount the volume named `todo-db` at the location specified, so that any files created at that location will still be available after the container stops.

#### Use Bind Mounts to create a development workflow
```bash
docker run -dp 3000:3000 \
    -w /app -v ${PWD}:/app \
    -v todo-db:/etc/todos \
    node:12-alpine \
    sh -c "yarn install && yarn run dev"
```
- `-w /app` - sets the "working directory" or the current directory that the command will run from.
- `-v ${PWD}:/app` - maps the current directory to the container location.
- Combine with mounting a named volume to also persist data.

```bash
docker logs -f <container-id>
$ nodemon src/index.js
[nodemon] 1.19.2
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] starting `node src/index.js`
Using sqlite database at /etc/todos/todo.db
Listening on port 3000
```
- Then, use `docker logs` to see log output from the running container.
- `-f` - Follow log output.
- In this case nodemon is watching for changes on the host machine and the app is restarted.

### Networks allow containers to communicate

> If two containers are on the same network, they can talk to each other. If they aren't, they can't.

```bash
docker network COMMAND
```

#### Create the network

```bash
docker network create todo-app
```

#### 

```bash
docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
```

- Docker will create a named volume for you automatically without creating one explicitly.

```bash
docker exec -it <mysql-container-id> mysql -p
```

- Verify that you can connect to the database.

```bash
docker run -dp 3000:3000 \
  -w /app -v ${PWD}:/app \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

- The run command has been updated to use the new mysql container for its database.
- The todo-app will use the mysql database because of this line in peristence/index.js: `if (process.env.MYSQL_HOST)`

### Docker Compose defines your application stack
https://docs.docker.com/compose/compose-file/

- A yaml file kept in the root of your project that can spin everything up or tear everything down with a single command.

```yaml
version: "3.8"
services:
  app:
    image: app:tag
networks:
volumes:
```

- Read over the documentation page or see examples, but this is the basic structure of the docker-compose.yml file.
- Each microservice is placed under the `services` heading.
- Allow services to communicate by adding entries under the `networks` heading.
- Persist data by creating a named volume under the `volumes` heading.

```bash
docker-compose up -d
docker-compose down
```
- Starting and stopping your application is now a single command.
- By default docker object names are prefixed with the current directory name.
- The containers and networks get removed when torn down.
- Supply the `--volumes` flag to also remove the volume.
- `-d` - Runs in the background.

```bash
docker-compose logs -f
```
- View logs when running in detached mode.


## 5. More documentation

https://docs.docker.com/

https://blog.jayway.com/2015/03/21/a-not-very-short-introduction-to-docker/