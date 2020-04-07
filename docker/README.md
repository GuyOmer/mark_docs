
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

	docker run --rm --it alpine

Boom! You just ran your first Docker Container, kill it by executing `exit`.
Lets break it down:

	docker run [OPTIONS] IMAGE [COMMAND] [ARGS]

**run -**  Run a command in a new container (more on command later).
**--rm -** Delete the container once it exists, good for housekeeping.
**-i -**  Interactive mode.
**-t -** Allocates a TTY.
**alpine -** [Alpine](https://hub.docker.com/_/alpine) is a minimal docker image with a basic Linux. Checkout [Docker Hub](https://hub.docker.com/search) for more pre-built images you can use out of the box.

You can re-run the command and play around in your container. Remember the container is your playgronud, you can do whatever you want without hard feelings.


## Doing Something Meaningful
Lets say we want a custom container, with a gcc compiler.
This time lets fire an ubuntu container, I trust you can manage this one yourself.
Unforuntly, ubuntu doesn't come pre-installed with gcc, so we will have to install it by ourselves, execute:

	sudo apt update && apt install build-essential -y
	
Lets make sure we have gcc installed:
`gcc -v`

Great!

Now let's compile something. Write a basic "Hello World" C code (note vim is not pre-installed), you can also use [super_complex.c](./super_complex.c) if you are lazy :chipmunk:.
Compile and run your code.
Neat right?

But what if we want to reuse this custom container?
Easy, let's just never exit it...

But I have a better solution, let's turn the creation of this container to a piece of code, and ship this code for whoever likes to see our "Hello World" Skills! (we can also commit this code, have it CR'ed and collbrate on it)

## Hello Dockefile

Dockerfiles are our way of turning the manual creation and configuration of a base imagic (alpine/ubuntu/...) to code. We can later ship this code to our users or fellow developers, and allow them to easily setup an identical environment to the one we are working on.
So lets start!
Create a new directory, and new file called Dockerfile inside it.
Dockerfiles consist of a very intuative and basic structure:

	INSTRUCTION arguments

Lets start by specificing the base image we will be coming from:

	FROM ubuntu
	
This will result in "ubuntu" being our base image, we can also specifiy a specific version by using "*ubuntu:20.04\ubuntu:18.04\ubuntu:focal*".

Now lets start re-constructings the steps we did in the former stage.
First, we want gcc, so we will use the RUN instruction to run a shell command:
	
	RUN apt update && apt install -y build-essential

The RUN command tells docker to execute a shell command using the arguments we supplied, so we actually executed:

	/bin/sh apt update && ...
	
To see if our Dockerfile works, we need to "buid" it, so execute:

	docker build -t myfirst .

This builds a Docker image from a Dockerfile and "context" (more on context in a moment).

	docker build [OPTIONS] PATH | URL | -

**-t -** Tag, this gives a name we later refrence this image.
**. (dot) -** This is the context. which can be either a PATH (current dir in our case), a URL or - (STDIN). Essentialy the context from where we refrence files from in all kind of instructions (this will be clearer in a moment). 

List all built images and check if you can find yours:

	docker images

Now run it:	

	docker run --rm -it myfirst
	
And check if gcc is installed (gcc -v).

But we dont want to re-write our complex C code again! Let use the context to put it in our image.
Exit the container, and come back to to Dockerfile, add the following:

	COPY hello.c /
	RUN make hello

The COPY instruction tells Docker to take hello.c (relative to the build context, which means ./hello.c) and place under / in the image.

Now build the image again and run it. You should see the *hello* executalbe under /.
 Taking things a step further, we can say that the sole purpose of *myfirst* image is to execute *hello*. To achive this we will use the CMD instruction:
	
	CMD /hello

Now when you rebuild and run the image, *hello* will be executed. Note -it flags are no longer relevant. If we want to run our image, and run a different command, we prepend it to our run command:

	docker run --rm -it myfirst /bin/bash

CMD is used to specificy the default command the image should run, and should usually be paired with an ENTRYPOINT instruction, but I won't go in-depth in this.

## Docker Compose

Docker Compose allows us to orchestrate multiple docker containers. I did not mention this in this guide, but for example you cannot state which ports your container exposes (the defualt is none) in a Dockerfile, so you might have a reverse proxy container (apache, nginx, traefik...) that you want to expose port 80, and then you have another machine the serves it's webservice on port 3000, so you want that one exposed as well. We can manually execute each machine (and use -p to expose ports), but this is aginanst the idea behind Docker. This is where docker compose comes in place, and allows us to orchesrate between containers, as opposed to only docker that "orchesrates" a single container.


## Tips and Tricks

Following these tricks will make you look like a docker professional.


### ADD v.s. COPY
Both can be used to place files from the build context, inside the image. We will prefer COPY as it is more explicit. ADD comes with magic features, like automatic tar extractions, fetch files from a URL and such. This features are cool and useful, but should be handeld with care.

A cool trick one can do. Is instead of having a ton of COPY, you can build your files hirecehy under some folder:

	root
	├── home
	│   └── user
	│       └── README.md
	└── opt
	    ├── gcc-9.1.0
	    │   ├── bin
	    │   ├── ...
	    └── some_toolchain


Then we will have a makefile creates a tarball from the directory, and builds the image.
In the docker file we will add:

	ADD root.tar.gz /

Docker will automatically unpack the tarball, and put all files where needed.
*Note this means we will use `make` for building, and not `docker build`.

### .dockerignore

Build context once again. As mentioned before, when issuing a docker build command. Docker takes the entire build context, archives it and sends it to the Docker daemon. This takes time! Think of having an enteire project in your directory, are you sure you want everything from you project directory in your image?
If the answer is *no*, then you should use a .dockerignore file. It's exactly like .gitignore and other ignore files. The .dockerignore file specificy which files should not be part of the build context, which allows us the minimize the build context.


### apt like a pro

### Watch the Cache
### Fine Layers

## Working offline
