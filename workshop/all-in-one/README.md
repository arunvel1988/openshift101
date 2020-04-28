# OpenShift Basics Quick Lab

Welcome to our OpenShift Basics quick lab!

**So what is OpenShift?**

OpenShift is an open source container application platform based on the Kubernetes container orchestrator for enterprise application development and deployment. In this workshop we'll be using an OpenShift cluster on IBM’s public cloud. OpenShift provides a way to empower developers to deploy code and not worry about the underlying ecosystem.

This workshop will show you a happy path to take advantage of most of the best parts of OpenShift and what it can offer. Specifically, this quick lab will show how a developer can use OpenShift to deploy a sample Node.js application.

**Let's get started!**

---

# Quick Lab: Deploy a Node.js application

In this exercise we'll deploy a Node.js application using OpenShift's Command Line Interface (CLI). We'll be using the ["Example Health"](https://github.com/IBM/node-s2i-openshift/) application.

From the editor's menu bar, choose the *Terminal* menu and click on *New Terminal*.

![Example Health details](https://raw.githubusercontent.com/IBM/openshift101/skills-network-ql/workshop/.gitbook/assets/snl-new-terminal.png)

## Clone the application's source code

First, clone the *Example Health* source code and change to that directory.

```bash
git clone https://github.com/IBM/node-build-config-openshift
cd node-build-config-openshift
```

{: codeblock}

> **NOTE** you may use the *Copy* button for these commands, but please remember to *Paste* them back in the terminal and hit *Enter*.

Let's start by taking a look at the `Dockerfile` in the application's root directory. A `Dockerfile` tells us how our application is being containerized, this is important in the next step as we'll be building an image to save and deploy in an OpenShift container. To view the `Dockerfile` you can use the `cat` command below or use the embedded IDE as shown in the image below.

```bash
cat Dockerfile
```

{: codeblock}

![Open Dockerfile](https://raw.githubusercontent.com/IBM/openshift101/skills-network-ql/workshop/.gitbook/assets/dockerfile.png)

Let's go through each line and read the corresponding comments.

```Dockerfile
# Use the official Node 10 image
FROM node:10

# Change directory to /usr/src/app
WORKDIR /usr/src/app

# Copy the application source code
COPY . .

# Change directory to site/
WORKDIR site/

# Install dependencies
RUN npm install

# Allow traffic on port 8080
EXPOSE 8080

# Start the application
CMD [ "npm", "start" ]
```

## Build a new image

Build your application's image by running the `oc new-build` command from your source code root directory. This will create a *Build* and an *ImageStream* of the app.

```bash
oc new-build --strategy docker --binary --docker-image node:10 --name example-health
```

{: codeblock}

The output should look like below:

```bash
oc new-build --strategy docker --binary --docker-image node:10 --name example-health
--> Found Docker image aa64327 (3 weeks old) from Docker Hub for "node:10"

    * An image stream tag will be created as "node:10" that will track the source image
    * A Docker build using binary input will be created
      * The resulting image will be pushed to image stream tag "example-health:latest"
      * A binary build was created, use 'start-build --from-dir' to trigger a new build

--> Creating resources with label build=example-health ...
    imagestream.image.openshift.io "node" created
    imagestream.image.openshift.io "example-health" created
    buildconfig.build.openshift.io "example-health" created
--> Success
```

Start a new build using the `oc start-build` command.

> **NOTE**: This command can take up to a minute to complete, please be patient!

```bash
oc start-build example-health --from-dir . --follow
```

{: codeblock}

The output should look like below:

```bash
oc start-build example-health --from-dir . --follow
Uploading directory "." as binary input for the build ...
.
Uploading finished
build.build.openshift.io/example-health-1 started
Receiving source from STDIN as archive ...
Replaced Dockerfile FROM image node:10
...
Successfully built 11bff161eb8e

Pushing image docker-registry.default.svc:5000/example-health-ns/example-health:latest ...
Pushed 0/12 layers, 17% complete
Pushed 1/12 layers, 42% complete
...
Pushed 11/12 layers, 100% complete
Pushed 12/12 layers, 100% complete
```

## Deploy the application

Now that we have our build, we can choose to deploy the application by running `oc new-app`. In this case we use the `-i` tag to indicate the image name.

```bash
oc new-app -i example-health
```

{: codeblock}

The output should look like below:

```bash
$ oc new-app -i example-health
--> Found image 11bff16 (8 minutes old) in image stream "example-health-ns/example-health" under tag "latest" for "example-health"

    * This image will be deployed in deployment config "example-health"
    * Port 8080/tcp will be load balanced by service "example-health"
      * Other containers can access this service through the hostname "example-health"
    * WARNING: Image "example-health-ns/example-health:latest" runs as the 'root' user which may not be permitted by your cluster administrator

--> Creating resources ...
    deploymentconfig.apps.openshift.io "example-health" created
    service "example-health" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/example-health'
    Run 'oc status' to view your app.
```

Expose the service using `oc expose`, a route will be created. This allows our application to be accessed from the outside world, it will give us a fully qualified URL to access our running application.

```bash
oc expose svc/example-health
```

{: codeblock}

Find the application's route by running `oc get routes`. This should return a fully qualified URL that we can access from any device.

```bash
oc get routes
```

{: codeblock}

The output should look like below:

```bash
$ oc get routes

NAME             HOST/PORT                                                                                                                        PATH      SERVICES         PORT       TERMINATION   WILDCARD
example-health   example-health-example-health-ns.aida-dev-apps-10-30-f2c6cdc6801be85fd188b09d006f13e3-0001.us-south.containers.appdomain.cloud             example-health   8080-tcp                 None
```

> The application's URL in the example above is `example-health-example-health-ns.aida-dev-apps-10-30-f2c6cdc6801be85fd188b09d006f13e3-0001.us-south.containers.appdomain.cloud`.

Copy the application's URL into a browser and login with the username and password `admin`:`test`.

![Example Health details](https://raw.githubusercontent.com/IBM/openshift101/skills-network-ql/workshop/.gitbook/assets/example-health-app.png)

---

# Quick Lab: Congratulations on deploying your first application on OpenShift

**Congratulations** on completing this quick lab, we hope you enjoyed it!

Here's a quick recap of what you did:

* Cloned a repository with sample code and a Dockerfile
* Built and pushed a new image to OpenShift's internal registry
* Deployed the application in a pod
* Exposed the app with a route

Before moving on to the next lab let's clean up our workspace by running these commands:

```
oc delete dc example-health
oc delete svc example-health
oc delete bc example-health
oc delete route example-health
oc delete imagestream example-health
```

{: codeblock}

Now that we're done cleaning up, move on to [Lab 2a]() to learn about OpenShift and Quarkus.
