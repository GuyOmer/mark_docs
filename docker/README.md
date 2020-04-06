---
id: docker
title: Docker
---

# Docker! :whale2:

I will go over the basics of using Docker, I will *not* be discussing how to setup a Docker Host, maintaining one and such. This is a power user quick guide to the world of Docker Containers.

## Environment
If you are online, use [play-with-docker](https://labs.play-with-docker.com) to experiment. Else setup a Docker Host by yourself, or Use an existing one.

## Just Run It!

Running your first docker container is easy!

```
docker run --rm --it alpine
```

Boom! You just ran your first Docker Container, kill it by executing `exit`.
Lets break it down:

`docker run [OPTIONS] IMAGE [COMMAND] [ARGS]`

**run - ** Run a command in a new container, in our case the command was /bin/ash which is alpine's shell.
**--rm - ** Delete the container once it exists, good for housekeeping.
**-i - ** Interactive mode.
**-t - ** Allocates a TTY.
**alpine - ** Alpine is a minimal docker image with basic linux stuff. Checkout [Docker Hub](https://hub.docker.com/search) for more pre-built images you can use out of the box.

You can re-run the command and play around in your container. Remember the container is your playgronud, you can do whatever you want without hard feelings.

## Doing Something Meaningful
Lets say we want a custom container, with a gcc compiler.
This time let's fire an ubuntu container, I trust you can manage this one yourself.
Unforuntly, ubuntu doesn't come pre-installed with gcc, so we will have to install it by ourselves, execute:
`sudo apt update && apt install build-essential -y`

Let's make sure we have gcc installed:
`gcc -v`

Great!

Now let's compile something. Write a basic "Hello World" C code (note vim is not pre-installed as well), you can also use super_complex.c if you are lazy :chipmunk:.
Compile and run your code.
Neat right?

But what if we want to reuse this custom container?
Easy, let's just never exit it...

But I have a better solution, let's turn the creation of this container to a piece of code, and ship this code for whoever likes to see our "Hello World" Skills!

## Hello Dockefile

Dockerfiles are our way of turning the manual creation and configuration of a base imagic (alpine/ubuntu/...) to code. We can later ship this code to our users or fellow developers, and allow them to easily setup an identical environment to the one we are working on.
So lets start!
Create a new directory, and new file called Dockerfile inside it.
Dockerfiles consist of a very intuative and basic structure:
`INSTRUCTION arguments`

Lets start by specificing the base image we will be coming from:
`FROM ubuntu`
This will result in "ubuntu" being our base image, we can also specifiy a specific version by using _"ubuntu:20.04\ubuntu:18.04\ubuntu:focal".

Now lets start constructings the steps we did in the former stage.
First, we want gcc, so we will use the RUN instruction to run a shell command:
`RUN apt update && apt install -y build-essential`

To see if our Dockerfile really works, we need to "buid" it, so execute:
```
docker build -t myfirst .
```

This builds a Docker image from a Dockerfile and "context" (more on context in a moment).

```
docker build [OPTIONS] PATH | URL | -
```

**-t - ** Tag, this gives a name we later refrence this image.
**. (dot) - ** This is the context (which can be either a PATH (current dir in our case), a URL or - (STDIN). Essentialy the context is the files you want your docker image be built with. For example if we were to have any other files in our directory, they would have been placed inside the image (current directory will be mapped to the image's root).

Now that our newly created image was built, lets run it:
`docker run --rm -it myfirst`
And check if gcc is installed (gcc -v).

But we dont want to re-write our complex C code again! Let use the context to put it in our image.
Quite the build


## Docker Compose
## Tips and Tricks
Following these tricks will make you look like a docker professional.
### ADD v.s. COPY
### apt like a pro
### Watch the Cache

## Working offline
