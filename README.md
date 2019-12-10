# Cloud Native Development (Windows 10 guide)

## Creating a Docker Image

Packs our application in a image with all the 'dependencies'

Check https://github.com/CloudNativeJS/docker for node.js docker file templates

1. Pull the 'Dockerfile' template:

> wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/Dockerfile

2. Pull the '.dockerignore' template:

> wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/.dockerignore

**NOTE: to use wget on Windows 10 you'll need to install it and add it to the OS environment variables, check out this guide on how to do this: https://www.addictivetips.com/windows-tips/install-and-use-wget-in-windows-10/**

Notice the Dockerfile created. What it does is installs any OS updates, copies the package.json file, does an npm install and only copies the . /app as late as possible

The .dockerignore file works in the same way as a .gitignore file. These are the files which are not going to be included in the docker image i.e are not required to execute.

## Building our docker image

To build our docker image we use the following command:

> docker build -t [nodeservername] -f [dockerfiletouse] .

-t specifices the tag (sever name) and -f specifies the docker file to use. Then we also had '.' which is the directory where we going to build

Now with a live example:

> docker build -t nodeserver -f Dockerfile .

Once created you can use the following command to check all the docker images you have:

> docker images

Finally we can run our application with:

> docker run -i -p 3000:3000 -t nodeserver

This maps port 3000 on the running docker container to port 3000 on our local machine.

Open http://localhost:3000 in your browser.

## Building a dev and debug Dockerfile

Still from https://github.com/CloudNativeJS/docker node.js docker file templates

1. Pull the 'Dockerfile' template:

   > wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/Dockerfile-tools

2. Then, get the run-dev script and the run-debug script:
   > wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/run-dev

> wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/run-debug

3. Edit the run-dev and run-debug scripts to make sure they are running our application: ./bin/www

Now build the Dockerfile-tools:

> docker build -t nodeserver-tools -f Dockerfile_tools .

This will build the tools image for our application

Now run:

> docker run -i -v %cd%/package.json:/tmp/package.json -v %cd%/node_modules:/tmp/node_modules -w /tmp -t node:10 npm install

to create the package dependencies inside the container

**NOTE: Make sure you have drive sharing turned on in the Docker settings. If you get a firewall error check https://stackoverflow.com/questions/42203488/settings-to-windows-firewall-to-allow-docker-for-windows-to-share-drive/43904051 for a solution**

Now we should be able to run dev and debug environments.

To run dev env:

> docker run -i -p 3000:3000 -v %cd%/:/app -v %cd%/node_modules:/app/node_modules -t nodeserver-tools /bin/run-dev
> This runs the application in the dev environment

To run debug env:

> docker run -i --expose 9229 -p 9229:9229 -p 3000:3000 -v %cd%/:/app -v %cd%/node_modules:/app/node_modules -t nodeserver-tools /bin/run-debug

To start debuging using Chrome, open a browser tab and type:

> chrome://inspect

## Building a production Dockerfile

Still from https://github.com/CloudNativeJS/docker node.js docker file templates

1. Pull the 'Dockerfile-run' template:

   > wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/Dockerfile-run

The Dockerfile-run is similar to the Dockerfile we used before but one crucial difference is that does a 2nd build with a node 10 slim image. The node 10 slim image only uses essencial packages/tools reducing the image size drastically. It also doesn't contain a compiler which makes it more secure.

Build the Dockerfile-run image:

> docker build -t nodeserver-run -f Dockerfile-run .

This will build an optimised run image. If you compare the images with 'docker images' you will see the 'nodeserver-run' and node '10-slim' builds are significantly smaller.

Now we can run our application inside the docker container as we did before using docker run:

> docker run -i -p 3000:3000 -t nodeserver-run

We now have exactly the same application running inside a docker container but this docker container is significantly smaller and it's been optimised to run in production environments.
