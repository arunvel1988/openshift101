# OpenShift 101 Lab

Welcome to our OpenShift 101 Lab

**So what is OpenShift?**

OpenShift is an open source container application platform based on the Kubernetes container orchestrator for enterprise application development and deployment. In this workshop we'll be using an OpenShift cluster on IBM’s public cloud. OpenShift provides a way to empower developers to deploy code and not worry about the underlying ecosystem.

This workshop will show you a happy path to take advantage of most of the best parts of OpenShift and what it can offer. Specifically, this quick lab will cover the following:

* Getting started with Kubernetes
* Deploying a Node.js application with Source-to-Image
* Debugging via logging, the terminal, and events
* Configuring our deployment by setting limits, scaling up, and using a config map

**Let's get started!**

---

# Exercise 0: Getting started with Kubernetes

A Kubernetes cluster consists of a set of worker machines, called nodes, that run **containerized** applications. The worker node(s) host the Pods that are the components of the application workload. The control plane manages the worker nodes and the Pods in the cluster.

## Checking access to a Kubernetes cluster

Check if the Kubernetes cluster is ready by running the command below to return the Kubernetes version.

```bash
kubectl version --short
```

{: codeblock}

## Deploying a microservices app on Kubernetes

The following command creates a [Kubernetes Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) called **my-app** from [`dewandemo/authors`](https://hub.docker.com/r/dewandemo/authors) image (which is on Docker Hub).

```bash
kubectl create deployment my-app --image=index.docker.io/dewandemo/authors:v1
```

{: codeblock}

Execute the following command to view the running pods:

```bash
kubectl get pods
```

{: codeblock}

## Creating replicas

One of the powerful feature of Kubernetes is the ability to scale your deployment up or down. The following command scales up **my-app** deployment to three [repliacas](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).

```bash
kubectl scale deployment my-app --replicas=3
```

{: codeblock}

Execute the following command to view the running pods:

```bash
kubectl get pods
```

{: codeblock}

You should see a list like the snippet below:

```bash
my-app-cf48574d-6cssh      1/1     Running   0          10m
my-app-cf48574d-c97hr      1/1     Running   0          9m4s
my-app-cf48574d-pqssh      1/1     Running   0          9m4s
```

## Self-healing of Kubernetes

Delete one of the pods:

```bash
kubectl delete pods my-app-cf48574d-6cssh
```

After few moments, execute the following command and you should still see three running pods.

```bash
kubectl get pods
```

{: codeblock}

Kubernetes deleted one of the pods but the deployment ensures a replica of three at all times so another pod was spun up.

## Updating the deployment to a newer image version

```bash
kubectl set image deployment/my-app authors=index.docker.io/dewandemo/authors:v2
```

{: codeblock}

This is how easily you can update a deployment to use a newer (or older) image.

## Clean up

```bash
kubectl delete deployments my-app
```

{: codeblock}

## Understanding What Happened

We covered the following:

* Pods were created from a publicly accessible image on DockerHub.
* We showed that pods are automatically re-created.
* Builds are pushed to OpenShift's internal regitry.
* You can use `kubectl scale deployment` to create replicas.
* You can use `kubectl set image` to deploy a new image version.

Now let's see how OpenShift can make this easier!

# Exercise 1: Deploy a Node application with Source-to-Image

In this exercise, you'll deploy a simple Node.js Express application - "Example Health". Example Health is a simple UI for a patient health records system. We'll use this example to demonstrate key OpenShift features throughout this workshop. You can find the sample application GitHub repository here: [https://github.com/IBM/node-s2i-openshift](https://github.com/IBM/node-s2i-openshift)

## Deploy a new application

Access your cluster's web console by clicking the `OpenShift Console` button on the top-right.

![Launch Console](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/snl-launch-console.png)

From the console's landing page, click on "+Add".

![Project Details](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/snl-project-details.png)

From the "Add" page, choose to add a new item "From Catalog".

![Catalog](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/developer-catalog.png)

Scroll down to the `Node.js` image. Click on that catalog button.

![NodeJS](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/developer-nodejs.png)

Click `Create Application`.

You'll see an form like this:

![Create Source-to-Image Application](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-gitrepo.png)

Enter the repository: `https://github.com/IBM/node-s2i-openshift`.

Then click the `Show Advanced Git Options`. and `/site` for the 'Context Dir'. Click 'Create' at the bottom of the window to build and deploy the application.

![Context Dir](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-context.png)

## Analyzing the build and deployment

Click on the center circle, you should see a build already in progress. You can click on the "View logs" to get more details.

![Build](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-build.png)

When the build has deployed, find the "Routes." Click on that link:

![Successful Build](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-success.png)

And you should see the login screen like the following:

![Login](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-login.png)

You can enter any strings for username and password, for instance `test:test` because the app is running in demo mode.

Congratulations! You've deployed a Node.js app to Kubernetes using OpenShift's Source-to-Image (S2I).

## Understanding What Happened

* [Source-to-Image](https://docs.openshift.com/container-platform/4.4/builds/build-strategies.html#build-strategy-s2i_build-strategies) (or S2I) is a framework that creates images from source code.
* The images are run as containers in pods.
* Creating a new application in OpenShift allows the user to specify multiple options for Building, Deploying, Scaling, Limits, and Routing.

<!--

# Exercise 2: Updating the application with a Webhook

So far we have been doing a lot of manual deployment. In a cloud-native world we want to move away from manual work and move toward automation. Wouldn't it be nice if our application rebuilt on git push events? Git webhooks are the way its done and openshift comes bundled in with git webhooks. Let's set it up for our project.

## Fork the code

To be able to setup git webhooks, we need to have elevated permission to the project. We don't own the repo we have been using so far. But since its opensource we can easily fork it and make it our own.

Fork the repo at [https://github.com/IBM/node-s2i-openshift](https://github.com/IBM/node-s2i-openshift)

![Fork](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/fork.png)

Now that I have forked the repo under my repo I have full admin privileges. As you can see I now have a settings button that I can change the repo settings with.

![Forked Repo](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/forked-repo.png)

We will come back to this page in a moment. Let's change our git source to our repo.

## Updating the build input

From our OpenShift dashboard for our project. Select `Builds`

![Build](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-build-config.png)

Select the `node-s-2-i-openshift` build. As of now this should be the only build on screen.

![Select Build](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-build-select.png)

Click on `Action` on the right and then select `Edit Build Config`

![Edit Build](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-build-edit.png)

Change lines `21` and `42` to the new Git Repository URL of our forked repository, and click `Save`.

![Save Build Config](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-build-save.png)

You will see this will not result in a new build. If you want to start a manual build you can do so by clicking `Start Build`. We will skip this for now and move on to the webhook part.

## Configuring a webhook

Click on the main `Overview` tab.

Scroll down and click `Copy URL with Secret` of the `GitHub Webook URL`.

![Copy GitHub Webhook](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/github-url-secret.png)

The webhook is in the structure

```text
https://c100-e.us-east.containers.cloud.ibm.com:31305/apis/build.openshift.io/v1/namespaces/example-health/buildconfigs/patientui/webhooks/<secret>/github
```

In our forked GitHub repo go to `Setting` > `Webhooks`. Then click `Add Webhook`

![Webhook Page](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/webhook-page.png)

In the Add Webhook page fill provide it with the following options:

* `Payload URL`: The copied URL from earlier
* `Content type`: Set to `application/json`
* `Secret`: Can remain empty (this is the default)
* `Events`: Just the **Push** event is fine for our use (this is the default)

Click on `Add webhook`

![Add Webhook](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/add-webhook.png)

> **NOTE**: If the webhook is reachable by GitHub you will see a green check mark.

Back in our OpenShift console we still would only see one build however. Because we added a webhook that sends us push events and we have no push event happening. Lets make one. The easiest way to do it is probably from the Github UI. Lets change some text in the login page.

Path to this file is `site/public/login.html` from the root of the directory. On Github you can edit any file by clicking the Pencil icon on the top right corner.

![Edit Page](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/edit-page.png)

Let's change the name our application to `Demo Health` (Line 21, Line 22). Feel free to make any other UI changes you feel like.

![Changes](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/changes.png)

Once done go to the bottom and click `commit changes`.

Go to the OpenShift build page again. This happens quite fast so you might not see the running state. But the moment we made that commit a build was kicked off.

![Running Build](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-rebuild-webhook.png)

In a moment it will show completed. Navigate to the main overview page to find the route.

![Routes](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-rebuild-overview.png)

> You could also go to `Applications > Routes` to find the route for the application.

If you go to your new route you will see your change.

![UI](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/nodejs-rebuild-updated.png)

-->

# Exercise 2: Logs, terminal, and events

In this exercise, we'll explore the out-of-the-box logging and events capabilities that are offered in OpenShift.

## Simulate Load on the Application

First, let's simulate some load on our application. Get the full URL of your application by running `oc get routes`:

```bash
oc get routes
```

Run the following script which will endlessly spam our app with requests. Open a new terminal and enter the following:

```bash
while sleep 1; do curl -s <your_app_route>/info; done
```

> **NOTE**: Ensure you added `/info` to the end!

## OpenShift Logging

Since we only created one pod, seeing our logs will be straight forward. Navigate to `View Logs` on the right on the main dashboard.

![Pods](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/view-logs.png)

You should be taken to something like the following. Scroll up and you should see the `DEBUG` like in the image. Scroll back down, and you should see a new line every second per the `curl` above.

![Logs](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/view-logs-details.png)

## OpenShift Terminal

One of the great things about Kubernetes is the ability to quickly debug your application pods with SSH terminals. This is great for development, but generally is not recommended in production environments. OpenShift makes it even easier by allowing you to launch a terminal directly in the dashboard.

Switch to the `Terminal` tab, and run the following commands.

```bash
# This command shows you the the project files.
ls
```

```bash
# This command shows you the running processes.
ps aux
```

![Terminal](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/terminal-output.png)

> **NOTE**: Try editing a file in the application source code to see what happens!

## OpenShift Events

When deploying new apps, making configuration changes, or simply inspecting the state of your cluster, OpenShift gives you an overview of your running assets.

You can also dive in a bit deeper - the `Events` tab is very useful for identifying the timeline of events and finding potential error messages.

![View Details](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/event-details.png)

You'll want to refer to this view throughout the lab. Almost all actions we take in in OpenShift will result in an event being fired in this view. As it is updated real-time, it's a great way to track changes to state.

## Understanding What Happened

* Application level logging via *Logging*
* Deployment level logging via *Events*
* Debugging by accessing the source code and running pods via *Terminal*

# Exercise 3: Configuring the deployment

In this exercise, we'll start to tweak our application's deployment.

## Switch to the Administrator view

Switch to the **Administrator** view to access additional configuration options about your cluster.

![Choose deployment](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/switch-to-admin.png)

## Select your deployment

Navigate to the **Workloads** > **Deployments** choice in the left-hand bar. Choose the `node-s-2-i-openshift` Deployment.

![Deployments](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/ocp-deployments.png)

## Enable Resource Limits

Setting resource limits on the pods running in our cluster allows us to define the maximum CPU and memory usage for a pod.

The sample application used consumes anywhere between **2m** - **20m** (millicores) of CPU and **25Mi** - **35Mi** MB of RAM.

Let's bump the limits to **30m** of CPU and **100Mi** of RAM by choosing the **YAML** tab.

![Limits YAML](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/ocp-limits-yaml.png)

In the YAML editor find the `template.spec.containers.resources` section (around line 44), and add the following limits.

```yaml
            resources:
              limits:
                cpu: 30m
                memory: 100Mi
              requests:
                cpu: 3m
                memory: 40Mi
```

> **NOTE**: Replace the entire `resources: {}` section. Ensure the spacing is correct. YAML uses strict indentation.

**Save** and **Reload** to see the new version.

Verify that the replication controller has been changed by navigating to **Events**.

![Resource Limits](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/ocp-dc-events.png)

## Scale up

Let's now scale up the number of pods in our deployment. From the *Overview* tab click the *Up* arrow a few times to scale the application to a few replicas.

![Scale up](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/increase-pods.png)

After a few moments, the *Overview* tab should show additional pods running. Click on the *Pods* tab to see additional details of the pods.

![Scale up](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/now-has-3-pods.png)

## Deleting a Pod

Remain on the *Pods* tab and let's delete one.

![Delete a pod](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/delete-a-pod.png)

Once a pod is deleted you'll see that a new pod is immediately re-created to take its place as defined by the number of replicas we have defined in our deployment.

## Create a ConfigMap

A **ConfigMap** is used to store data in key-value pairs. Pods can consume **ConfigMaps** as environment variables, command-line arguments, or as configuration files in a volume.

A **ConfigMap** allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.

Navigate to the **Workloads** > **Config Maps** choice in the left-hand bar. Choose to create a new one by clicking the **Create Config Map** button.

![Config Map menu](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/config-map-menu.png)

Give it a `name` and `data`, like the example below. **Your NAMESPACE will be different!**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: node-s-2-i-openshift
  namespace: sn-labs-stevemar
data:
  API_KEY: abc-123
```

![Create a Config Map](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/create-config-map.png)

## Apply the Config Map to your deployment

A config map in and of itself is not very useful. It must be applies to a deloyment. Let's go back to ours.

Navigate to the **Workloads** > **Deployments** choice in the left-hand bar. Choose the `node-s-2-i-openshift` Deployment.

From the **Environment** tab choose to apply the `node-s-2-i-openshift` Config Map by selecting it from the drop down list.

![Choose to apply the Config Map](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/choose-config-map.png)

Quickly switch back to the **Pods** tab to see the status of the pods.

![Pods re-spinning](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/config-map-pods.png)

If you switched over quickly enough you'll be able to see the original pods terminating and new pods being re-created. **Why**? Applying a new configuration (like applying a new config map) triggers a re-deployment.

## Go to the terminal of a pod

Choose any of the pods that are up and running.

![Select a pod](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/select-a-pod.png)

From the **Terminal** tab run the command:

```bash
env | grep API
```

To see the values of the **ConfigMap** you just applied.

![See ConfigMap values](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/pod-terminal-config-map.png)

## Understanding What Happened

* Started to configure our deployment
* Added limits to each pod
* Scaled up pods, observed self-healing
* Began to leverage environment variables via Config Maps

<!--

## NOT ENABLED in SNL yet

## Enable Autoscaler

Now that we have resource limits, let's enable autoscaler.

By default, the autoscaler allows you to scale based on CPU or Memory. The UI allows you to do CPU only \(for now\). Pods are balanced between the minifmum and maximum number of pods that you specify. With the autoscaler, pods are automatically created or deleted to ensure that the average CPU usage of the pods is below the CPU request target as defined.
In general, you probably want to start scaling up when you get near `50`-`90`% of the CPU usage of a pod. In our case, let's make it `1`% to test the autoscaler since we are generating minimal load.

1. Navigate to **Workloads > Horizontal Pod Autoscalers**, then hit **Create Horizontal Pod Autoscaler**.

![HPA](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/ocp-hpa.png)

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: patient-hpa
  namespace: example-health
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: patient-ui
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: 1
```

Hit **Create**.

## Test Autoscaler

If you're not running the script from the [previous exercise](exercise-2.md#simulate-load-on-the-application), the number of pods should stay at 1.

Check by going to the **Overview** page of **Deployments**.

![Scaled to 1 pod](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/ocp-hpa-before.png)

Start simulating load by hitting the page several times, or running the script. You'll see that it starts to scale up:

![Scaled to 4/10 pods](https://raw.githubusercontent.com/IBM/openshift101/skills-network/workshop/.gitbook/assets/ocp-hpa-after.png)

That's it! You now have a highly available and automatically scaled front-end Node.js application. OpenShift is automatically scaling your application pods since the CPU usage of the pods greatly exceeded `1`% of the resource limit, `30` millicores.

-->

# Congratulations on completing the lab!

**Congratulations** on completing this lab, we hope you enjoyed it!

Here's a quick recap of what was covered:

* Getting started with Kubernetes
* Deploying a Node.js application with Source-to-Image
* Debugging via logging, the terminal, and events
* Configuring our deployment by setting limits, scaling up, and using a config map

Before moving on to the next lab let's clean up our workspace by running these commands:

```bash
oc delete deployment node-s-2-i-openshift
oc delete svc node-s-2-i-openshift
oc delete bc node-s-2-i-openshift
oc delete route node-s-2-i-openshift
oc delete imagestream node-s-2-i-openshift
oc delete configmap node-s-2-i-openshift
oc delete secret node-s-2-i-openshift-generic-webhook-secret
oc delete secret node-s-2-i-openshift-github-webhook-secret
```

{: codeblock}
