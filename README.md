# Java Docker React Bonus Material

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

## Dockerising a React App

Now we're going to look at how we can put one of our React Apps into a Docker container and run it from there. Let's create and run a new React App using Vite, that we'll then use as the basis for our Docker Image. Run the following command to create a new `react` app called `react-docker-example-app` using `vite`:

```bash
npm create vite@latest react-docker-example-app -- --template react
```

Next move into the directory created by this using:

```bash
cd react-docker-example-app
```

then download and install the dependencies:

```bash
npm install
```

and finally run the application to make sure everything is working:

```bash

npm run dev
```

This will start a development server running on Port 5173 by default. We can change this to a different port number in the `vite.config.js` file in the root of our project. Use `Ctrl-C` or just press `q` whilst the terminal is in focus, to stop the dev server from running and open `vite.config.js` using VS Code, it will probably look like this:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
})
```

Change that to:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  server: {
   host: true,
   port: 8000,
    watch: {
      usePolling: true
    }
 }
})
```

and then rerun the application. This time the development server will run on Port 8000. We can deploy our development server version to a Docker container and we might do so if we wanted to work on it, using the container without necessarily installing the same versions of Node and other dependencies. If we want a version of our application that we can deploy into production this would not be what we wanted however. Instead we would want to create our production version of our application and then deploy that into the container. Let's look at how we can do each of these.

## Run the Dev Server in a Docker Container

To do this we are going to copy our project into a suitable image and run the container that results, when it starts up we'll get it to run the dev server using port 8000, and then access that from outside the container. **REMEMBER: THIS IS NOT WHAT WE WILL DO WHEN DEPLOYING OUR SITE IN PRODUCTION!!!!!**

To generate the image we're going to create a `Dockerfile` file in the top level of our project and then use that to download the base image, copy the files from the project into it and specify the other required commands that will run our application when we run the container. Create a new text file in the root directory of our project called `Dockerfile`.

Add the following contents to it (we'll go through what they mean line by line afterwards):

```bash
FROM node:18-alpine

WORKDIR /react-docker-example-app/

COPY public/ /react-docker-example-app/public
COPY src/ /react-docker-example-app/src
COPY package.json /react-docker-example-app/
COPY vite.config.js /react-docker-example-app/
COPY index.html /react-docker-example-app/

RUN npm install

CMD ["npm", "run", "dev"]
```

The first line:

```bash
FROM node:18-alpine
```

downloads the base image we're using for this image which is based on Node version 18, the alpine version gives us a very lightweight base image to build upon

The line which starts `WORKDIR` specifies the working directory that we will be in when we start the container running.

Then we have 4 lines which use `COPY` to copy various directories and files into that working directory inside the container.

The line:

```bash
RUN npm install
```

does the same thing in our container that happens in the local filesystem when we run `npm install`, it looks at the `package.json` file and installs all of the dependencies it finds there. This time however it does it into the container rather than our local machine.

The final line:

```bash
RUN ["npm", "run", "dev"]
```

contains the command that gets run in our working directory when the container is run (ie `npm run dev`).

Once you've saved the `Dockerfile` make sure your terminal is open in the top level directory and run the following command to build the image:

```bash
docker build -t react-docker-example-app .
```

This may take a while to run depending on what you have installed previously, after a few minutes hopefully you'll see the success message.

At this stage we are ready to run the container and access our react app through the browser. Make sure you've stopped any other copies of the app from running and then run the following:

```bash
docker run -dp 8000:8000 --name react-docker react-docker-example-app:latest
```

If everything worked properly you should be able to access the application at [http://localhost:8000](http://localhost:8000)

## Run the Production Version of the Application in a Docker Container

The Dev server that is packaged with the application in React is fine for light use in a development setting but should not be deployed to a production setting, so we need to use `vite` to generate the static files our application is built on and then package them into a container along with lightweight server that is suitable for production use. A common server to use in this situation is called `Nginx` which has its own Docker image we can make use of as the basis for our production version of the application. Before we can do this though we need to generate the production files for our application. Run the following command to create these:

```bash
npm run build
```

This will create a new directory called `dist` in the directory and add in the required files. This folder is what we need our `Nginx` server to serve (we don't need to download and install all of the node modules as anything required has already been copied across into our code). As we already have a `Dockerfile` in here we can either rewrite that one to use our new commands or make a new folder with a new `Dockerfile` in it and copy the `dist` folder into there. The docker build command would then need to be run from that folder and if we altered our react application at all, we would need to rebuild it and copy the new folder over to there too. I'm going to rewrite the existing `Dockerfile` and work from the top level folder.

Change the content of the `Dockerfile` to have the following in it:

```bash
FROM nginx

COPY dist /usr/share/nginx/html
```

Build the container based on the `Dockerfile` as we did previously (give it a different name though).

```bash
docker build -t nginx-react .
```

Then run it exposing Port 80 in the container. That way we can just navigate to [http://localhost](http://localhost) to access the application:

```bash
docker run -dp 80:80 --name nginx-react nginx-react
```

Experiment with this and see how you could develop this further to use backends based on Spring or that use a PostgreSQL Database.