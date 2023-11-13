# Java Docker Day 1 workshop

## Learning Objectives

- Install and setup docker
- Create a simple docker container
- Run the container and interact with it
- Make a more complex container
- Interact with that container
- Create a simple React Project and run it from a container

## Getting Setup with Docker

- Windows users follow these instructions: [https://docs.docker.com/desktop/install/windows-install/](https://docs.docker.com/desktop/install/windows-install/)
- Mac users follow these instructions: [https://docs.docker.com/desktop/install/mac-install/](https://docs.docker.com/desktop/install/mac-install/)

On Windows you will probably need to install WSL (Windows Subsystem for Linux) and may need to change some BIOS settings to enable virtualisation to happen, this is normal.

## Allow Docker to Run from GitBash in Windows

On Mac you should find that Docker runs fine in the terminal, on Windows we will need to jump through a few more hoops to enable this, all of the rest of the exercies assume that you are going to interact with Docker from the shell/terminal. Follow these steps to enable this:

1. We need to open the Docker Desktop app first.
2. Then click on the cog at the top right to open the `Settings` menu.
3. Go to the `Resources` section from the left hand menu.
4. Select `WSL Integration` from the sub-menu that appears and select Ubuntu and any other WSL Linux versions you want to be able to run docker from.
5. Then open your WSL command prompt and type `docker --version` to make sure it has installed.

## Hello World Docker Style

Open you rterminal and run `docker run hello-world` this will download the `hello-world` Docker image and run it. You should see something like the following:

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

This is a way to check that everything installed correctly and works as expected.

## A Slightly More Complex Docker Image/Container

In Docker terms, we have base `images` that we either create or download, we then customise them to some extent to create a `container` (based on the image) which then run. The Docker Hub contains a large number of pre-configured images that we can work with as well as containers that people and projects have shared. Let's start by using a common base image that uses the `Busybox` tool to provide basic command line tools. If you've ever done any work with embedded systems or trying to get command line tools to run on Android you've probably come across `Busybox` it is a tool which when installed replicates a lot of common command line tools for bash inside a single binary. We can create/download a docker container which has Busybox installed and ready to go really easily.

Run the following command:

```bash
docker pull busybox
```

This will `pull` a docker image which contains `Busybox` from the `Docker Registry` and save it onto our machine.

We can run it by typing:

```bash
docker run busybox
```

But this won't have any visible effect.

Try:

```bash
docker run busybox echo "Well hello there."
```

and you should see the message output to the shell.

Running:

```bash
docker ps
```

will show you any running docker images:

```bash
docker ps -a
```

will show you previously running ones and:

```bash
docker images
```

will show you the base images currently downloaded onto your machine.

To start an interactive `busybox` session we can do:

```bash
docker run -it busybox sh
```

and then type various bash commands in the resulting prompt to experiment. When you're done type `exit` to exit the session.

We can remove the previous sessions by running:

```bash
docker container prune
```

## Using Somebody Else's Image

We can use existing images from the Docker Registry provided we know their name. Here's a static-site one that features in a reasonably popular tutorial about docker, we can download and run it all in a single step like this:

```bash
docker run --rm -it prakhar1989/static-site
```

The `--rm` option will automatically remove the session when we finish running it. It may take a little while to download the first time you run it, but once it has been downloaded it will check if a newer version exists and just run the local copy from then onwards. If successful you'll see:

```bash
Nginx is running...
```

in the terminal but no indication of how to reach the site that is being served. To actually get the site accessible to us we need to tell docker which port to use. Hit `Ctrl-C` to stop the container running. Then do the following to run the container again:

```bash
docker run -d -P --name static-site prakhar1989/static-site
```

The `-d` option detaches the terminal and the container from each other. `-P` assigns random numbers to any available ports and `--name` gives the container a human readable label we can use to access information about it. We can find out the ports that were assigned by doing:

```bash
docker port static-site
```

We can then go to `localhost` followed by the port number to access the site that's being served. (The one that port 80 maps to will probably work, the one that port 443 maps to probably won't as there is no certificate associated with it.)

Run the following to stop it running:

```bash
docker stop static-site
```

You can also specify which port to use when running the application by doing:

```bash
docker run -d -p 4000:80 --name static-site prakhar1989/static-site
```

Sometimes to access the resulting site I had to connect to `0.0.0.0:4000` rather than `localhost:4000` I am not sure why, if `localhost` doesn't work then you can try that instead.
