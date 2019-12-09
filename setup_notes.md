# Creating a Docker Image

Packs our application in a image with all the 'dependencies'

Check https://github.com/CloudNativeJS/docker for node.js docker file templates

1. Pull the 'Dockerfile' template:

> wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/Dockerfile'

2. Pull the '.dockerignore' template:

> wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/.dockerignore'

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
