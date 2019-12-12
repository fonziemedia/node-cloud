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

## Tagging and version control

So far our images have been tagged with 'latest' by default.

We should use a major, minor, fix versioning convention to tag our images i.e 1.0.0

We use the 'docker tag' command to tag our images. Let's give our run image a version of 1.0.0:

> docker tag nodeserver-run nodeserver:1.0.0
> if you run 'docker images' again you'll see the image id is exactly the same as our nodeserver-run image. Our tag is actually just a tag that points to the same image.

### Publishing to docker.hub

Use 'docker login' to log in to docker hub on the command line

We will need to rename our image to have the name and the tag of the image to match our username, so we'll need to re-reun our docker tag:

> docker tag nodeserver-run 11914901/nodeserver:1.0.0

Now we can use docker push to push our image to docker.hub:

> docker push 11914901/nodeserver:1.0.0
> Our image will now be hosted and available to other people with access to your namespace to use.

## Usefull commands

Delete a local image:

> docker rmi [imagename]

Pull from docker.hub:

> docker pull [imagename]

Push to docker.hub:

> docker login ;
> docker push [imagename]

Run an image:

> docker run -i -p 3000:3000 -t [imagename]

Build an image from a docker file:

> docker build -t [imagename] -f [Dockerfilename] .

To see all local images/tags:

> docker images

## Deploying to Kubernetes

Kubernetes will let us run multiple instances of the same container and load balance any incoming request across the multiple instances. It will also allow us to connect to other things. That might be something as simple as a database or it might be another application or microservice.
And Kubernetes will also let us to integrate with cloud capabilities. Things like authentication, monitoring and logging.

Kubernetes can be installed locally on your desktop in multiple ways. The easiest way of doing that is using Docker for desktop. To do that we can go to our Docker logo in our taskbar and click on settings > Kubernetes > Enable Kubernetes (Also make sure you allocate enough memory/cpu to run Kubernetes. You can do this from the Advanced menu).

As well as installing Kubernetes, it's also installed the Kubernetes CLI. The Kubernetes CLI is is called **kubectl**.

We now need to take our application and deploy it to Kubernetes. To do that, we're going to use Helm. Helm describes itself as helping you define, install and upgrade even the most complex Kubernetes application. Helm provides us with the ability to package our app, along with all of the configuration it needs, into a chart. That chart then describes how our app should be deployed and run. In order to install Helm itself, we can follow the [Docs](https://helm.sh/docs/intro/install/) on its website. On Windows you can use [chocolatey](https://chocolatey.org/docs/installation).

As well as controlling deployments, Helm has a sense of repos. We can type Helm repo to see the commands that are available for us working with repositories. We can use this to list repos, and these are all of the places that Helm currently knows about to receive pre-existing charts, including the stable repo.

In helm3 you'll need to setup the stable repo manually using the following command:

> helm repo add stable https://kubernetes-charts.storage.googleapis.com/

There are actually a number of repositories that are available for you to use, and we can see those by going to Helm Hub. [Helm Hub](https://hub.helm.sh/) is a search engine that Helm itself provides that lets you find charts to install existing software.

Whilst these are pre-existing software, we want to package our app. To do that, we're going to go to a set of chart templates that are provided by [CloudNativeJS](https://www.cloudnativejs.io/). Here we can go on Deploy to Kubernetes, click on View on GitHub, and this takes us to a project which provides a template Helm chart. Download, unzip and copy the chart directory to your project folder.

Next, open chart/nodeserver/**values.yaml** to configure it.

Change the repository to your docker repository image that your using i.e 11914901/nodeserver and check the tag also matches the version

Now we have a chart that's ready to take our application and deploy it to Kubernetes.

To do that we're going to use the helm CLI:

> helm install nodeserver chart/nodeserver

This is now going to take our docker image and deploy it to Kubernetes and (not on Windows) start everything running.

To see the status of our deployment we can use **helm status** and use the name of our deployment which was node server:

> helm status nodeserver

This now shows us that we have a pod which is running. Pod is the Kubernetes name for an instance, and this shows that we have one instance of our node server running.

To get this running on Windows you'll need to port-forward your deployment and map it to port 3000:

> kubectl port-forward deployment/[DEPLOYMENT_NAME] [PORT_NUMBER]:3000

You can also port-forward and individual deployment:

> kubectl port-forward [PODNAME] [PORT_NUMBER]:3000

That deployed a single instance of our application and that's controlled by this value in the **values.yaml** in the Helm chart which is replicate which was set to one.

Before we can change that let's go to the CLI and actually delete our existing deployment, let's first go to our project directory and then type:

> Helm delete --purge nodeserver

Now on our **values.yaml** file, change the replicaCount value to 2 and hit Save. Reinstall the app as done previously. Now you should have 2 pods instead of 1. Restart the service again and you should now have 2 instances running.

We now have two instances running, but ideally we should have three or more. This gives us what's called **triple redundancy**. Which is ideal for resilience and availability. Because should one instance fail there are two others which are able to share the load from the failed instance, to do that let's go back to our project and change our replicaCount to three, let's change that to three and hit save.

See also **seamless upgrade** using the Helm upgrade command:
> helm upgrade nodeserver chart/nodeserver

As well as allowing you to upgrade Helm also allows us to roll back to previous revisions, to see that happening we can first to **helm history** to see the list of revisions. This shows us we've got a couple of revisions we can move back to. Let's go ahead and do that by doing **helm rollback nodeserver 1**

This is successfully rolled back and we can see that by typing **helm status**

So we can use upgrade and rollback to seamlessly change from one state to another state without losing any uptime of our application.