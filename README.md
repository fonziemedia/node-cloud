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

Follow the instructions given to get the IP and PORT numbers of your application. In Windows PowerShell we would do:
>$SAMPLE_NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services nodeserver-service)
>
>$SAMPLE_NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
>

then:
>start http://${SAMPLE_NODE_IP}:${SAMPLE_NODE_PORT}

or:
>start http://localhost:${SAMPLE_NODE_PORT}

You can also use port-fowarding to map it to a specific port:

> kubectl port-forward deployment/[DEPLOYMENT_NAME] [PORT_NUMBER]:3000

That deployed a single instance of our application and that's controlled by the **replicaCount** value in the **values.yaml** in the Helm chart which is set to one by default.

Before we can change that let's go to the CLI and actually delete our existing deployment, let's first go to our project directory and then type:

> Helm delete --purge nodeserver

Now on our **values.yaml** file, change the replicaCount value to 2 and hit Save. Reinstall the app as done previously. Now you should have 2 pods instead of 1. Restart the service again and you should now have 2 instances running.

We now have two instances running, but ideally we should have three or more. This gives us what's called **triple redundancy**. Which is ideal for resilience and availability. Because should one instance fail there are two others which are able to share the load from the failed instance, to do that let's go back to our project and change our replicaCount to three, let's change that to three and hit save.

See also **seamless upgrade** using the Helm upgrade command:
> helm upgrade nodeserver chart/nodeserver

As well as allowing you to upgrade Helm also allows us to roll back to previous revisions, to see that happening we can first to **helm history** to see the list of revisions. This shows us we've got a couple of revisions we can move back to. Let's go ahead and do that by doing **helm rollback nodeserver 1**

This is successfully rolled back and we can see that by typing **helm status**

So we can use upgrade and rollback to seamlessly change from one state to another state without losing any uptime of our application.

## Adding Support for Health

As well as providing template docker files and helm charts to simplify packaging and deploying Node.js apps to Kubernetes. [Cloud Native JS](https://cloudnativejs.io/) also provides Node modules for adding health checks to an application. This is all done in the [health checking](https://github.com/CloudNativeJS/cloud-health-connect) project. 

This provides a connect middleware for adding health checks, so liveness and readiness end points to an existing application. To install it we can use npm:
> npm install @cloudnative/health-connect

Now that we have it in our dependencies, we need to require it into our application, and we can do that in app.js:

### Seting up a HealthChecker

> const health = require('@cloudnative/health-connect');
>
>let healthcheck = new health.HealthChecker();

### Adding a PingChecker

> let pingcheck = new health.PingCheck("example.com");
>
> healthcheck.registerLivenessCheck(pingcheck);

Register a separate Liveness endpoint:
> app.use('/live', health.LivenessEndpoint(healthcheck));

Now that we've done that, our application should make a request to that post name, when our liveness end point is hit. Let's restart our application and go to our browser and see our end point. When we refresh, we can now see that we have a registered check, that does a ping check against example.com/80. Now try disconnecting the network on your browser tools to see what happens.

### Adding a readiness endpoint

>app.use("/ready", health.ReadinessEndpoint(healthcheck));

Now let's change our Ping Checker to be done on readyness check instead

> healthcheck.registerReadinessCheck(pingcheck);

Finally, before we can deploy this into Kubernetes, we need to rebuild the Docker image to include our new code changes. To do that, we'll use **docker build** to build an image called nodeserver-run using the Dockerfile-run Dockerfile as before. But as what we're deploying has been re-tagged, we also need to re-tag our Docker image. We do that using **docker tag**. And this time, rather than using a version of 1.0.0 we're going to increase that to 1.1.0 because we have a feature update.

So we now have a fully running application inside kubernetes that has started to leverage the power of the platform itself. By adding our own custom liveness and readiness checks we can make it possible to tell kubernetes when to restart and when to stop sending load to us. And that affects the status of ready, and whether a re-start happens for our application.

## Add support for metrics

### Prometheus
Prometheus is a metrics system that collects data from both Kubernetes itself and from any Prometheus-enabled application

As with a number of the other components, [CloudNativeJS](https://www.cloudnativejs.io/) provides support for adding Prometheus-style metrics to your application.

Here what's provided is an npm module called appmetrics-prometheus, which collects application metrics and makes it available to Prometheus.

**Note: On Windows, before you can install appmetrics-prometheus you'll need have Python installed (you can use Chocolatey to install it) and the windows-build-tools npm package ("npm install --global --production windows-build-tools") **

Once all the above pre-requisites are met:
>npm install appmetrics-prometheus

Next we need to include that into app.js. To do that, we go to the top of our file, and we require in the module:
>var prom = require('appmetrics-prometheus').attach();

That's all we need to do. This has required the module into our application, and told it to attach to the Express server that it finds, and this is then going to expose a /metrics endpoint which Prometheus will use to collect data.

Now we restart our application, we should have a **/metrics** end point. 

This provides a lot of statistical data about our application. This isn't particularly human-readable, but it doesn't need to be, because Prometheus is going to gather this data and use it to display graphs and charts that you're going to create yourself. 

Update our Docker image to include these changes using **docker build**. Also, remember to tag it as we've added a new feature: 1.2.0.

Finally, we need to remember to update our chart to use the 1.2.0 version, and we do that inside the chart directory in our **values.yaml**. We save that change, and now we can redeploy our application into Kubernetes to have the metrics running live in Kubernetes.

### Deploy Prometheus to Kubernetes
We still, however, need to install Prometheus into our Kubernetes cluster. Luckily, Helm Hub already has support for Prometheus. So, we can go to Helm Hub, and search the charts for Prometheus:
> helm install --name prometheus stable/prometheus --version 9.5.2 --namespace prometheus

Then follow the post-install instructions given or in Windows:
>$POD_NAME=$(kubectl get pods --namespace prometheus -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
>
>kubectl --namespace prometheus port-forward $POD_NAME 9090

Now you'll be able to see our prometheus server on http://localhost:9090/

### Deploy Grafana to Kubernetes

> helm install --name grafana stable/grafana --namespace grafana --set adminPassword=PASSWORD

Use the post-install instructions. Or on Windows:
>$POD_NAME=$(kubectl get pods --namespace grafana -l "app=grafana,release=grafana" -o jsonpath="{.items[0].metadata.name}")
>
>kubectl --namespace grafana port-forward $POD_NAME 3000

This is now forwarding requests to local host 3000 through to our Grafana server. So let's open the browser and go to local host 3000. And here's our Grafana dashboard. Now, it'll ask us to log in and we set the admin password to PASSWORD. This now logs us in to Grafana.

Start by a adding a data source which will be Prometheus as we've already installed it and is up and running. So we can click on the Prometheus box. Now, this requires us to put in a URL for where to find the Prometheus deployment:
> prometheus.server.prometheus.svc.cluster.local.

The first of that Prometheus server is the name of our deployment, and the second Prometheus was the namespace that we deployed it into. So this is why we gave Prometheus a namespace is also to make it easier to name it here.

Save and Test, which will save the configuration and test the Prometheus as reachable. This shows us that our data source is working and it's successfully connected to the Prometheus data store. Next, we're going to want to be able to create a dashboard. Now, Grafana itself provides a community of dashboards that are already available. So you can go to the Grafana website and search for various different types of dashboards. And we're going to use one of those to import, to bootstrap ourselves and have a ready-made Kubernetes dashboard. To do that, we click on the plus button and hit Import. We'll save our changes and then we'll enter an ID of a dashboard. Here we'll use 1621, which we could find from the dashboard page. And we'll hit Load. This is a Kubernetes cluster monitoring dashboard that has been provided as part of the community. Change the Prometheus setting to use our Prometheus data store and hit Import.

This now sets up a ready-made Kubernetes dashboard that shows us lots of information about our Kuberenetes install itself. It can show us how much memory is being used, how much CPU is being used, how much file system is being used.

### Build charts for your applications

Now we're going to extend that dashboard to add some custom chart specifically for our Node.js application that we've deployed. To do that, we're first going to go to the top and select the Add panel button. This lets us add new charts based either on a query first or on choosing a visualization first. Let's click on Add Query. We're going to start by adding the same query that we used when we were just using Prometheus. And that is os_cpu_used_ratio. It auto completes that for us so we can select os_cpu_used_ratio and when we do that, it starts to chart the CPU data that it has for each of our Node server instances. We also have the option of limiting the chart to specific applications or charts or instances by using any of these fields that it shows you in the line. So, we can take our value of os_cpu_used_ratio and make sure that it's specifically for our Node server application by adding an extra filter. And the filter we're going to use is kubernetes_name equals nodeserver-service. 






