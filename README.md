# Docker bootstrapper
Docker is oriented to applications, but we usually want a set of services launched in our container when it is started. E.g. you want to have an apache2 server with a php application in it, and it needs a mysql server.
You'd probably need to create a custom script for your entrypoint that starts all those services.

This project is intended to make a bootstrapping procedure like the well-known _init_ mechanism in Linux.

Using this bootstrapper, the scripts that are insider the folder _/etc/docker-boot/conf.d_ will be executed in order at boot time, and the container will be kept in the _/bin/bash_ command.

## Using it
You should add the following lines to your _Dockerfile_ file.

```bash
# -------------------------------------------------------------------
# Install the bootstrapper
# -------------------------------------------------------------------

ADD ./start-node /opt/docker-boot/
ADD ./conf.d /etc/docker-boot/conf.d/
ENTRYPOINT [ "/opt/docker-boot/start-node" ]
```

and build your docker image like this:

```bash
$ docker build -t ubuntu:bootstrap .
```

Then you are done and you can start your container issuing a command like this one:
```bash
$ docker run --rm -it ubuntu:bootstrap bash
```

## Warning
Everytime you modify the content of _conf.d_, you'll need to re-create the container image.

## Example

### Container with SSH server

You can use a Dockerfile like _Dockerfile.ssh_ included in this repository to create a SSH server. Notice that it is included the file _02SSH_ in folder _/etc/docker-boot/conf.d_:

```bash
FROM ubuntu
MAINTAINER Carlos de Alfonso <caralla@upv.es>

# -------------------------------------------------------------------
# Setup the ssh server
# -------------------------------------------------------------------

RUN apt-get update && apt-get install -y openssh-server
EXPOSE 22
RUN useradd -m -s /bin/bash ubuntu
RUN echo 'ubuntu:ubuntu' | chpasswd

# -------------------------------------------------------------------
# Install the bootstrapper
# -------------------------------------------------------------------

ADD ./start-node /opt/docker-boot/
ADD ./conf.d /etc/docker-boot/conf.d/
ADD ./conf.d/examples/02SSH /etc/docker-boot/conf.d/
ENTRYPOINT [ "/opt/docker-boot/start-node" ]
```

Then you can build the docker image

```bash
docker build -f Dockerfile.ssh -t ubuntu:ssh .
```

And finally you'll be able to launch your new SSH server in a container:

```bash
docker run -id ubuntu:ssh
```