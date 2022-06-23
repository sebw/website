In this lab, we're going to deploy the web component of the ParksMap application which is also called `parksmap` and uses OpenShift's service discovery mechanism to discover the backend services deployed and shows their data on the map.

![Application architecture](/images/roadshow-app-architecture-parksmap-1.png)

## Exercise: Deploying your First Image

Let's start by doing the simplest thing possible - get a plain old
Docker-formatted image to run on OpenShift. This is incredibly simple to do.
With OpenShift it can be done directly from the web console.

Return to the [Web Console](http://console-openshift-console.{{cluster_subdomain}}/k8s/cluster/projects).

If you're no longer on the Developer perspective, return there now. 

From the left menu, click *+Add*. You will see a screen where you have multiple options to deploy application to OpenShift. Click *Container Image* to open a dialog that will allow you to specify the information for the image you want to deploy.

![Add from Container Image](/images/parksmap-devconsole-container-image.png)

In the *Image Name* field, copy/paste the following into the box:

```
quay.io/openshiftroadshow/parksmap:latest
```

OpenShift will then go out to the container registry specified and interrogate the image.

Your screen will end up looking something like this:

![Explore Project](/images/parksmap-image.png)

In *Runtime Icon* you can select the icon to use in OpenShift Topology View for the app. You can leave the default OpenShift icon, or select one from the list.

NOTE: The purpose of this exercise is to deploy a microservice from an agnostic existing container image (Frontend, this was made with Spring Boot). The specific programming language path you have chosen is described and implemented in the next microservice chapter (Backend).

Make sure to have the correct values in:

* *Application Name* : workshop
* *Name* : parksmap

Ensure *Deployment* is selected from *Resource* section.

*Un-check* the checkbox next to *Create a route to the application*. For learning purposes, we will create a *Route* for the application later in the workshop.

At the bottom of the page, click *Labels* in the Advanced Options section and add some labels to better identify this deployment later. Labels will help us identify and filter components in the web console and in the command line.

We will add 3 labels. After you enter the name=value pair for each label, press *tab* or de-focus with mouse before typing the next. First the name to be given to the application.

```
app=workshop
```

Next the name of this deployment.

```
component=parksmap
```

And finally, the role this component plays in the overall application.

```
role=frontend
```

![Deploy image](/images/parksmap-image-options.png)

Next, click the blue *Create* button. You will be directed to the *Topology* page, where you should see the visualization for the `parksmap` deployment config in the `workshop` application.

![Topology View with Parksmap](/images/parksmap-dc-topology.png)

These few steps are the only ones you need to run to get a
container image deployed on OpenShift. This should work with any
container image that follows best practices, such as defining an EXPOSE
port, not needing to run specifically as the *root user* or other user name, and a single non-exiting CMD to execute on start.

>NOTE: Providing appropriate labels is desired when deploying complex applications for organization purposes.   
>OpenShift uses a label *app* to define and group components together in the Overview page. OpenShift will create this label with some default if the user doesn't provide it explicitly.

### Background: Containers and Pods

Before we start digging in, we need to understand how containers and *Pods* are
related. We will not be covering the background on these technologies in this lab but if you have questions please inform the instructor. Instead, we will dive right in and start using them.

In OpenShift, the smallest deployable unit is a *Pod*. A *Pod* is a group of one or more OCI containers deployed together and guaranteed to be on the same host.
From the official OpenShift documentation:

>Each *Pod* has its own IP address, therefore owning its entire port space, and
>containers within pods can share storage. *Pods* can be "tagged" with one or
>more labels, which are then used to select and manage groups of *pods* in a
>single operation.


*Pods* can contain multiple OCI containers. The general idea is for a *Pod* to
contain a "main process" and any auxiliary services you want to run along with that process. Examples of containers you might put in a *Pod* are, an Apache HTTPD
server, a log analyzer, and a file service to help manage uploaded files.

## Exercise: Examining the Pod

If you click on the `parksmap` entry in the Topology view, you will see some information about that deployment config. The *Resources* tab may be displayed by default. If so, click on the *Details* tab. 

![Details Tab image](/images/switchtoresources.png)

On that panel, you will see that there is a single *Pod* that was created by your actions.

![Pod overview](/images/parksmap-overview.png)

You can also get a list of all the *Pods* created within your *Project*, by navigating to *Workloads -> Pods* in the Administrator perspective of the web console.

![Pod list](/images/parksmap-podlist.png)

This *Pod* contains a single container, which
happens to be the `parksmap` application - a simple Spring Boot/Java application.

You can also examine *Pods* from the command line:

```
oc get pods
```

You should see output that looks similar to:

```
NAME                READY   STATUS      RESTARTS   AGE
parksmap-65c4f8b676-k5gkk    1/1     Running     0          20s
```

The above output lists all of the *Pods* in the current *Project*, including the
*Pod* name, state, restarts, and uptime. Once you have a *Pod*'s name, you can
get more information about the *Pod* using the `oc get` command.  To make the
output readable, I suggest changing the output type to *YAML* using the
following syntax:

>NOTE: Make sure you use the correct *Pod* name from your output.


```
oc get pod parksmap-65c4f8b676-k5gkk -o yaml
```

You should see something like the following output (which has been truncated due
to space considerations of this workshop manual):


```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    k8s.v1.cni.cncf.io/network-status: |-
      [{
          "name": "",
          "interface": "eth0",
          "ips": [
              "10.131.0.93"
          ],
          "default": true,
          "dns": {}
      }]
    k8s.v1.cni.cncf.io/networks-status: |-
      [{
          "name": "",
          "interface": "eth0",
          "ips": [
              "10.131.0.93"
          ],
          "default": true,
          "dns": {}
      }]
    openshift.io/generated-by: OpenShiftWebConsole
    openshift.io/scc: restricted
  creationTimestamp: "2021-01-05T17:00:32Z"
  generateName: parksmap-65c4f8b676-
  labels:
    app: parksmap
    component: parksmap
    deploymentconfig: parksmap
    pod-template-hash: 65c4f8b676
    role: frontend
```

The web interface also shows a lot of the same information on the *Pod* details
page. If you click on the name of the *Pod*, you will
find the details page. You can also get there by clicking on the `parksmap` deployment config on the *Topology* page, selecting *Resources*, and then clicking the *Pod* name.

![Parksmap Resources](/images/parksmap-dc-resources.png)

From here you can see configuration, metrics, environment variables, logs, events and get a Terminal shell on the running pod.

![Pod Details](/images/parksmap-pod.png)

![Pod Events](/images/parksmap-pod-events.png)

Getting the `parksmap` image running may take a little while to complete. Each
OpenShift node that is asked to run the image has to pull (download) it, if the
node does not already have it cached locally. You can check on the status of the
image download and deployment in the *Pod* details page, or from the command
line with the `oc get pods` command that you used before.

The default view in the *Developer* console is *Graph View*. You can switch between *Graph* and *List* views by using the toggle in the top right of the console.

![List View Toggle](/images/nationalparks-listview.png)

![Topology View Toggle](/images/nationalparks-graphview.png)

### Background: Customizing the Image Lifecycle Behavior

Whenever OpenShift asks the node's CRI (Container Runtime Interface) runtime (Docker daemon or CRI-O) to run an image, the runtime will check to make sure it has the right "version" of the image to run.
If it doesn't, it will pull it from the specified registry.

There are a number of ways to customize this behavior. They are documented in
https://{{DOCS_URL}}/applications/creating_applications/creating-applications-using-cli.html#applications-create-using-cli-image_creating-applications-using-cli[specifying an image]
as well as
https://{{DOCS_URL}}/openshift_images/managing_images/image-pull-policy.html[image pull policy].

### Background: Services

*Services* provide a convenient abstraction layer inside OpenShift to find a
group of similar *Pods*. They also act as an internal proxy/load balancer between
those *Pods* and anything else that needs to access them from inside the
OpenShift environment. For example, if you needed more `parksmap` instances to
handle the load, you could spin up more *Pods*. OpenShift automatically maps
them as endpoints to the *Service*, and the incoming requests would not notice
anything different except that the *Service* was now doing a better job handling
the requests.

When you asked OpenShift to run the image, it automatically created a *Service*
for you. Remember that services are an internal construct. They are not
available to the "outside world", or anything that is outside the OpenShift
environment. That's okay, as you will learn later.

The way that a *Service* maps to a set of *Pods* is via a system of *Labels* and
*Selectors*. *Services* are assigned a fixed IP address and many ports and
protocols can be mapped.

There is a lot more information about
https://{{DOCS_URL}}/architecture/understanding-development.html#understanding-kubernetes-pods[Services],
including the YAML format to make one by hand, in the official documentation.

Now that we understand the basics of what a *Service* is, let's take a look at
the *Service* that was created for the image that we just deployed. In order to
view the *Services* defined in your *Project*, enter in the following command:


```
oc get services
```

You should see output similar to the following:


```
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
parksmap   ClusterIP   172.30.22.209  <none>        8080/TCP   3h
```

In the above output, we can see that we have a *Service* named `parksmap` with an
IP/Port combination of 172.30.22.209/8080TCP. Your IP address may be different, as
each *Service* receives a unique IP address upon creation. *Service* IPs are
fixed and never change for the life of the *Service*.

In the Developer perspective from the *Topology* view, service information is available by clicking the `parksmap` deployment config, then *Resources*, and then you should see the `parksmap` entry in the *Services* section.

![Services list](/images/parksmap-serviceslist.png)

You can also get more detailed information about a *Service* by using the
following command to display the data in YAML:


```
oc get service parksmap -o yaml
```

You should see output similar to the following:


```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    openshift.io/generated-by: OpenShiftWebConsole
  creationTimestamp: "2020-09-30T14:10:12Z"
  labels:
    app: workshop
    app.kubernetes.io/component: parksmap
    app.kubernetes.io/instance: parksmap
    app.kubernetes.io/part-of: workshop
    component: parksmap
    role: frontend
  name: parksmap
  namespace: user1
  resourceVersion: "1062269"
  selfLink: /api/v1/namespaces/user1/services/parksmap
  uid: e1ff69c8-cb2f-11e9-82a1-0267eec7e1a0
spec:
  clusterIP: 172.30.22.209
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: parksmap
    deploymentconfig: parksmap
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

Take note of the `selector` stanza. Remember it.

Alternatively, you can use the web console to view information about the *Service* by clicking on it from the previous screen.

![Service](/images/parksmap-service.png)

It is also of interest to view the YAML of the *Pod* to understand how OpenShift
wires components together. For example, run the following command to get the
name of your `parksmap` *Pod*:


```
oc get pods
```

You should see output similar to the following:


```
NAME                        READY   STATUS    RESTARTS   AGE
parksmap-65c4f8b676-k5gkk   1/1     Running   0          5m12s
```

Now you can view the detailed data for your *Pod* with the following command:


```
oc get pod parksmap-65c4f8b676-k5gkk -o yaml
```

Under the `metadata` section you should see the following:


```yaml
  labels:
    app: parksmap
    deploymentconfig: parksmap
```

* The *Service* has `selector` stanza that refers to `deploymentconfig=parksmap`.
* The *Pod* has multiple *Labels*:
** `app=parksmap`
** `deploymentconfig=parksmap`

*Labels* are just key/value pairs. Any *Pod* in this *Project* that has a *Label* that
matches the *Selector* will be associated with the *Service*. To see this in
action, issue the following command:


```
oc describe service parksmap
```

You should see something like the following output:


```
Name:              parksmap
Namespace:         user1
Labels:            app=workshop
                   app.kubernetes.io/component=parksmap
                   app.kubernetes.io/instance=parksmap
                   app.kubernetes.io/part-of=workshop
                   component=parksmap
                   role=frontend
Annotations:       openshift.io/generated-by: OpenShiftWebConsole
Selector:          app=parksmap,deploymentconfig=parksmap
Type:              ClusterIP
IP:                172.30.22.209
Port:              8080-tcp  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.128.2.90:8080
Session Affinity:  None
Events:            <none>
```

You may be wondering why only one endpoint is listed. That is because there is
only one *Pod* currently running.  In the next lab, we will learn how to scale
an application, at which point you will be able to see multiple endpoints
associated with the *Service*.

--- 
## Exercise: Scaling and Self Healing

### Background: Deployments and ReplicaSets

While *Services* provide routing and load balancing for *Pods*, which may go in and
out of existence, *ReplicaSet* (RS) and *ReplicationController* (RC) are used to specify and then
ensure the desired number of *Pods* (replicas) are in existence. For example, if
you always want your application server to be scaled to 3 *Pods* (instances), a
*ReplicaSet* is needed. Without an RS, any *Pods* that are killed or
somehow die/exit are not automatically restarted. *ReplicaSets* and *ReplicationController* are how OpenShift "self heals" and while *Deployments* control *ReplicaSets*, *ReplicationController* here are controlled by *DeploymentConfigs*.

From the https://{{DOCS_URL}}/applications/deployments/what-deployments-are.html[deployments documentation]:

>Similar to a replication controller, a ReplicaSet is a native Kubernetes API object that ensures a specified number of pod replicas are running at any given time. The difference between a replica set and a replication controller is that a replica set supports set-based selector requirements whereas a replication controller only supports equality-based selector requirements.

In Kubernetes, a *Deployment* (D) defines how something should be deployed. In almost all cases, you will end up using the *Pod*, *Service*,
*ReplicaSet* and *Deployment* resources together. And, in
almost all of those cases, OpenShift will create all of them for you.

There are some edge cases where you might want some *Pods* and an *RS* without a *D*
or a *Service*, and others, so feel free to ask us about them after the labs.

## Exercise: Exploring Deployment-related Objects

Now that we know the background of what a *ReplicaSet* and
*Deployment* are, we can explore how they work and are related. Take a
look at the *Deployment* (D) that was created for you when you told
OpenShift to stand up the `parksmap` image:

```
oc get deployment
```

```
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
parksmap   1/1     1            1           20m
```

To get more details, we can look into the *ReplicaSet* (*RS*).

Take a look at the *ReplicaSet* (RS) that was created for you when
you told OpenShift to stand up the `parksmap` image:

```
oc get rs
```

```
NAME                  DESIRED   CURRENT   READY   AGE
parksmap-65c4f8b676   1         1         1       21m
```

This lets us know that, right now, we expect one *Pod* to be deployed
(`Desired`), and we have one *Pod* actually deployed (`Current`). By changing
the desired number, we can tell OpenShift that we want more or less *Pods*.

OpenShift's *HorizontalPodAutoscaler* effectively monitors the CPU usage of a
set of instances and then manipulates the RCs accordingly.

You can learn more about the CPU-based
[Horizontal Pod Autoscaler here](https://{{DOCS_URL}}/nodes/pods/nodes-pods-autoscaling.html#nodes-pods-autoscaling-about_nodes-pods-autoscaling)

## Exercise: Scaling the Application

Let's scale our parksmap "application" up to 2 instances. We can do this with
the `scale` command. You could also do this by incrementing the Desired Count in the OpenShift web console. Pick one of these methods; it's your choice.

```
oc scale --replicas=2 deployment/parksmap
```

You can also scale up to two pods in the *Developer Perspective*. From the Topology view, first click the `parksmap` deployment config and select the *Details* tab:

![Details](/images/parksmap-details.png)

Next, click the *^* icon next to the Pod visualization to scale up to 2 pods.

![Scaling up](/images/parksmap-scaleup.png)

To verify that we changed the number of replicas, issue the following command:


```
oc get rs
```


```
NAME                  DESIRED   CURRENT   READY   AGE
parksmap-65c4f8b676   2         2         2       23m
```

You can see that we now have 2 replicas. Let's verify the number of pods with
the `oc get pods` command:


```
oc get pods
```


```
NAME                        READY   STATUS    RESTARTS   AGE
parksmap-65c4f8b676-fxcrq   1/1     Running   0          92s
parksmap-65c4f8b676-k5gkk   1/1     Running   0          24m
```

And lastly, let's verify that the *Service* that we learned about in the
previous lab accurately reflects two endpoints:


```
oc describe svc parksmap
```

You will see something like the following output:


```
Name:              parksmap
Namespace:         user1
Labels:            app=workshop
                   app.kubernetes.io/component=parksmap
                   app.kubernetes.io/instance=parksmap
                   app.kubernetes.io/part-of=workshop
                   app.openshift.io/runtime-version=latest
                   component=parksmap
                   role=frontend
Annotations:       openshift.io/generated-by: OpenShiftWebConsole
Selector:          app=parksmap,deploymentconfig=parksmap
Type:              ClusterIP
IP:                172.30.136.210
Port:              8080-tcp  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.128.2.138:8080,10.131.0.93:8080
Session Affinity:  None
Events:            <none>
```

Another way to look at a *Service*'s endpoints is with the following:


```
oc get endpoints parksmap
```

And you will see something like the following:


```
NAME       ENDPOINTS                           AGE
parksmap   10.128.2.90:8080,10.131.0.40:8080   45m
```

Your IP addresses will likely be different, as each pod receives a unique IP
within the OpenShift environment. The endpoint list is a quick way to see how
many pods are behind a service.

You can also see that both *Pods* are running in the Developer Perspective:

![Scaled](/images/parksmap-scaled.png)

Overall, that's how simple it is to scale an application (*Pods* in a
*Service*). Application scaling can happen extremely quickly because OpenShift
is just launching new instances of an existing image, especially if that image
is already cached on the node.

## Application "Self Healing"

Because OpenShift's *RSs* are constantly monitoring to see that the desired number
of *Pods* actually are running, you might also expect that OpenShift will "fix" the
situation if it is ever not right. You would be correct!

Since we have two *Pods* running right now, let's see what happens if we
"accidentally" kill one. Run the `oc get pods` command again, and choose a *Pod*
name. Then, do the following:


```
oc delete pod parksmap-65c4f8b676-k5gkk && oc get pods
```

```
pod "parksmap-65c4f8b676-k5gkk" deleted
NAME                        READY   STATUS    RESTARTS   AGE
parksmap-65c4f8b676-bjz5g   1/1     Running   0          13s
parksmap-65c4f8b676-fxcrq   1/1     Running   0          4m48s
```

Did you notice anything? One container has been deleted, and there's a new container already being created. 

Also, the names of the *Pods* are slightly changed.
That's because OpenShift almost immediately detected that the current state (1
*Pod*) didn't match the desired state (2 *Pods*), and it fixed it by scheduling
another *Pod*.

Additionally, OpenShift provides rudimentary capabilities around checking the
liveness and/or readiness of application instances. If the basic checks are
insufficient, OpenShift also allows you to run a command inside the container in
order to perform the check. That command could be a complicated script that uses
any installed language.

Based on these health checks, if OpenShift decided that our `parksmap`
application instance wasn't alive, it would kill the instance and then restart
it, always ensuring that the desired number of replicas was in place.

More information on probing applications is available in the
[Application
Health](https://{{DOCS_URL}}/applications/application-health.html) section of the documentation and later in this guide.

## Exercise: Scale Down

Before we continue, go ahead and scale your application down to a single
instance. Feel free to do this using whatever method you like.

>sWARNING: Don't forget to scale down back to 1 instance your `parksmap` component as otherwise you might experience some weird behavior in later labs. This is due to how the application has been coded and not to OpenShift itself.

--- 
## Exposing Your Application to the Outside World

In this lab, we're going to make our application visible to the end users, so they can access it.

![Application architecture](/images/roadshow-app-architecture-parksmap-2.png)

### Background: Routes

While *Services* provide internal abstraction and load balancing within an
OpenShift environment, sometimes clients (users, systems, devices, etc.)
**outside** of OpenShift need to access an application. The way that external
clients are able to access applications running in OpenShift is through the
OpenShift routing layer. And the data object behind that is a *Route*.

The default OpenShift router (HAProxy) uses the HTTP header of the incoming
request to determine where to proxy the connection. You can optionally define
security, such as TLS, for the *Route*. If you want your *Services*, and, by
extension, your *Pods*, to be accessible from the outside world, you need to
create a *Route*.

## Exercise: Creating a Route

You may remember that when we deployed the `parksmap` application, we un-checked the checkbox to create a *Route*. Normally it would have been created for us automatically. Fortunately, creating a *Route* is a pretty straight-forward process. You simply `expose` the *Service* via the command line or via the *Administrator Perspective*.

### Creating a route using the Web Console

In the *Administrator Perspective* click *Networking -> Routes* and then the *Create Route* button. 

Insert *parksmap* in *Name* field.

From *Service* field, select *parksmap*. For *Target Port*, select *8080*.

In *Security* section, check *Secure route*. Select *Edge* from *TLS Termination* list.

Leave all other fields blank and click *Create*:

![Create Route1](/images/parksmap-route-create-1.png)
![Create Route2](/images/parksmap-route-create-2.png)

When creating a *Route*, some other options can be provided, like the hostname and path for the *Route* or the other TLS configurations.

### Creating a route using the command line

When using the command line, we can first verify that we don't already have any existing *Routes*:

```
oc get routes
```

```
No resources found.
```

Now we need to get the *Service* name to expose:

```
oc get services
```

```
NAME       CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
parksmap   172.30.169.213   <none>        8080/TCP   5h
```

Once we know the *Service* name, creating a *Route* is a simple one-command task:

```
oc create route edge parksmap --service=parksmap
```

```
route.route.openshift.io/parksmap exposed
```

Verify the *Route* was created with the following command:

```
oc get route
```

```
NAME       HOST/PORT                            PATH      SERVICES   PORT       TERMINATION   WILDCARD
parksmap   parksmap-{{project_namespace}}.{{cluster_subdomain}}             parksmap   8080-tcp   edge          None
```

You can also verify the *Route* in the *Developer Perspective* under the *Resources* tab for your `parksmap` deployment configuration. Also note that there is a decorator icon on the `parksmap` visualization now. If you click that, it will open the URL for your *Route* in a browser.

![Route created](/images/parksmap-route-created.png)

This application is now available at the URL shown in the Developer Perspective. Click the link and you will see it.

NOTE: At first time, the Browser will ask permission to get your position. This is needed by the Frontend app to center the world map to your location, if you don't allow it, it will just use a default location.

![Empty map](/images/parksmap-route-empty-map.png)

---
# Deploying Java Code

In this section, we're going to deploy a backend service, developed in Java that will expose 2 main REST endpoints to the visualizer
application (`parksmap` web component that was deployed in the previous labs).
The application will query for national parks information (including its
coordinates) that is stored in a MongoDB database.  This application will also
provide an external access point, so that the API provided can be directly used
by the end user.

![Application architecture](/images/roadshow-app-architecture-nationalparks-1.png)

### Background: Source-to-Image (S2I)

In a previous lab, we learned how to deploy a pre-existing image
image. Now we will expand on that by learning how OpenShift builds
container images using source code from an existing repository.  This is accomplished using the Source-to-Image project.

[Source-to-Image (S2I)](https://github.com/openshift/source-to-image) is a
open source project sponsored by Red Hat that has the following goal:


>Source-to-image (S2I) is a tool for building reproducible container images. S2I
>produces ready-to-run images by injecting source code into a container image and
>assembling a new container image which incorporates the builder image and built
>source.   
>The result is then ready to use with docker run. S2I supports
>incremental builds which re-use previously downloaded dependencies, previously
>built artifacts, etc.


OpenShift is S2I-enabled and can use S2I as one of its build mechanisms (in
addition to building container images from Dockerfiles, and "custom" builds).

OpenShift runs the S2I process inside a special *Pod*, called a Build
Pod, and thus builds are subject to quotas, limits, resource scheduling, and
other aspects of OpenShift.

A full discussion of S2I is beyond the scope of this class, but you can find
more information about it either in the
[OpenShift S2I documentation](https://{{DOCS_URL}}/openshift_images/using_images/using-s21-images.html)
or on [GitHub](https://github.com/openshift/source-to-image). The only key concept you need to
remember about S2I is that it's magic.

## Exercise: Creating a Java application

The backend service that we will be deploying as part of this exercise is
called `nationalparks`.  This is a Java Spring Boot application that performs 2D
geo-spatial queries against a MongoDB database to locate and return map
coordinates of all National Parks in the world. That was just a fancy way of
saying that we are going to deploy a webservice that returns a JSON list of
places.

### Add to Project
Because the `nationalparks` component is a backend to serve data that our
existing frontend (parksmap) will consume, we are going to build it inside the existing
project that we have been working with. To illustrate how you can interact with OpenShift via the CLI or the web console, we will deploy the nationalparks component using the web console.

### Using Application Code on an Embedded Git Server

OpenShift can work with any accessible Git repository. This could be GitHub,
GitLab, or any other server that speaks Git. You can even register webhooks in
your Git server to initiate OpenShift builds triggered by any update to the
application code!

The repository that we are going to use is already cloned in the internal Gogs repository
and located at the following URL:

Your Gogs credentials are:

```
username: {{username}}
password: {{GOGS_PASSWORD}}
```

[Gogs Repository](http://gogs-{{INFRA_PROJECT}}.{{cluster_subdomain}}/{{username}}/nationalparks.git)

Later in the lab, we want you to make a code change and then rebuild your
application. This is a fairly simple Spring framework Java application.

### Build the Code on OpenShift

Similar to how we used *+Add* before with an existing image, we
can do the same for specifying a source code repository. Since for this lab you
have your own git repository, let's use it with a simple Java S2I image.

In the Developer Perspective, click *+Add* in the left navigation, go to the *Git Repository* section and then choose *From Git* option.

![Add to Project](/images/nationalparks-show-add-options.png)

The *Import from Git* workflow will guide you through the process of deploying your app based on a few selections.

Enter the following for Git Repo URL:

```
http://gogs-{{INFRA_PROJECT}}.{{cluster_subdomain}}/{{username}}/nationalparks.git
```

In *Git Type* select *Other*.

>NOTE: if you copied the Gogs URL correctly (check spaces etc) and you still get a warning like 'Git repository is not reachable' please ignore it at this time, since the UI may fail to ping the repo sometimes.

Select *Java* as your Builder Image, and be sure to select version *openjdk-11-ubi8* to have OpenJDK 11.

![Import from Git](/images/nationalparks-import-from-git-url-builder.png)

NOTE: All of these runtimes shown are also made available via *Templates* and
*ImageStreams*, which will be discussed in a later lab.

Scroll down to the *General* section. Select:

* *Application Name* : workshop
* *Name* : nationalparks


In *Resources* section, select *Deployment*.

Expand the Labels section and add 3 labels:

The name of the Application group:

```
app=workshop
```

Next the name of this deployment.

```
component=nationalparks
```

And finally, the role this component plays in the overall application.

```
role=backend
```

![Runtimes](/images/nationalparks-configure-service.png)

Click *Create* to submit.

To see the build logs, in Topology view, click the `nationalparks` entry, then click on *View Logs* in the *Builds* section of the *Resources* tab.

![Nationalparks build](/images/nationalparks-java-new-java-build.png)


This is a Java-based application that uses Maven as the build and dependency system.  For this reason, the initial build
will take a few minutes as Maven downloads all of the dependencies needed for
the application. You can see all of this happening in real time!

From the command line, you can also see the *Builds*:

```
oc get builds
```

You'll see output like:

```
NAME              TYPE      FROM          STATUS     STARTED              DURATION
nationalparks-1   Source    Git@b052ae6   Running    About a minute ago   1m2s
```

You can also view the build logs with the following command:

```
oc logs -f builds/nationalparks-1
```

After the build has completed and successfully:

* The S2I process will push the resulting image to the internal OpenShift registry
* The *Deployment* (D) will detect that the image has changed, and this
  will cause a new deployment to happen.
* A *ReplicaSet* (RS) will be spawned for this new deployment.
* The RS will detect no *Pods* are running and will cause one to be deployed, as our default replica count is just 1.

In the end, when issuing the `oc get pods` command, you will see that the build Pod
has finished (exited) and that an application *Pod* is in a ready and running state:

```
NAME                    READY     STATUS      RESTARTS   AGE
nationalparks-1-tkid3   1/1       Running     3          2m
nationalparks-1-build   0/1       Completed   0          3m
parksmap-57df75c46d-xltcs        1/1       Running     0          2h
```

If you look again at the web console, you will notice that, when you create the
application this way, OpenShift also creates a *Route* for you. You can see the
URL in the web console, or via the command line:

```
oc get routes
```

Where you should see something like the following:

```
NAME            HOST/PORT                                                   PATH      SERVICES        PORT       TERMINATION       WILDCARD
nationalparks   nationalparks-{{ project_namespace  }}.{{cluster_subdomain}}             nationalparks   8080-tcp
parksmap        parksmap-{{ project_namespace  }}.{{cluster_subdomain}}                  parksmap        8080-tcp        edge        none
```

In the above example, the URL is:

```
http://nationalparks-{{ project_namespace  }}.{{cluster_subdomain}}
```

Since this is a backend application, it doesn't actually have a web interface.
However, it can still be used with a browser. All backends that work with the parksmap
frontend are required to implement a `/ws/info/` endpoint. To test, visit this URL in your browser:

[National Parks Info Page](http://nationalparks-{{project_namespace}}.{{cluster_subdomain}}/ws/info/)

>WARNING: The trailing slash is *required*. If the Pod is Running and the application is not available, please wait a few seconds and retry since we haven't configured yet Health Checks for that.

You will see a simple JSON string:

```
{"id":"nationalparks","displayName":"National Parks","center":{"latitude":"47.039304","longitude":"14.505178"},"zoom":4}
```

Earlier we said:

```
This is a Java Spring Boot application that performs 2D geo-spatial queries
against a MongoDB database
```

But we don't have a database. Yet.



--- 

## Adding a Database (MongoDB)

In this section we will deploy and connect a MongoDB database where the
`nationalparks` application will store location information.

Finally, we will mark the `nationalparks` application as a backend for the map
visualization tool, so that it can be dynamically discovered by the `parksmap`
component using the OpenShift discovery mechanism and the map will be displayed
automatically.

![Application architecture,800,align="center"](/images/roadshow-app-architecture-nationalparks-2.png)

### Background: Storage

Most useful applications are "stateful" or "dynamic" in some way, and this is
usually achieved with a database or other data storage. In this lab we are
going to add MongoDB to our `nationalparks` application and then rewire it to
talk to the database using environment variables via a secret.

We are going to use the MongoDB image that is included with OpenShift.

By default, this will use *EmptyDir* for data storage, which means if the *Pod*
disappears the data does as well. In a real application you would use
OpenShift's persistent storage mechanism to attach real-world storage (NFS,
Ceph, EBS, iSCSI, etc) to the *Pods* to give them a persistent place to store their data.

### Background: Templates

In this module we will create MongoDB from a *Template*, which is useful mechanism in OpenShift to define parameters for certain values, such as
DB username or password, that can be automatically generated by OpenShift at
processing time.

Administrators can load *Templates* into OpenShift and make them available to
all users. Users can create *Templates* and load them
into their own *Projects* for other users (with access) to share and use.

The great thing about *Templates* is that they can speed up the deployment
workflow for application development by providing a "recipe" of sorts that can
be deployed with a single command.  Not only that, they can be loaded into
OpenShift from an external URL, which will allow you to keep your templates in a
version control system. 


## Exercise: Instantiate a MongoDB Template

In this step we will create a MongoDB template inside our project, so that is only visible to our user and we can access it from Developer Perspective to create a MongoDB instance.

```
oc create -f https://raw.githubusercontent.com/openshift-labs/starter-guides/ocp-4.8/mongodb-template.yaml -n {{project_namespace}}
```

What just happened? What did you just `create`? The item that we passed to the `create`
command is a *Template*. `create` simply makes the template available in
your *Project*.

## Exercise: Deploy MongoDB

As you've seen so far, the web console makes it very easy to deploy things onto
OpenShift. When we deploy the database, we pass in some values for configuration.
These values are used to set the username, password, and name of
the database.

The database image is built in a way that it will automatically configure itself
using the supplied information (assuming there is no data already present in the
persistent storage!). The image will ensure that:

- A database exists with the specified name
- A user exists with the specified name
- The user can access the specified database with the specified password

In the Developer Perspective in your `{{ project_namespace }}` project,
click *+Add* and go to *Developer Catalog* section and click to *Database*. 
In the Databases view, you can click *Mongo* to filter for just MongoDB.

![Data Stores](/images/nationalparks-databases-catalog-databases.png)

Alternatively, you could type `mongodb` in the search box. Once you have drilled down to see MongoDB, find the *MongoDB (Ephemeral)* template and select it.  You will notice that there are multiple
MongoDB templates available.  We do not need a database with persistent storage, so the ephemeral Mongo
template is what you should choose.  Go ahead and select the ephemeral template and click the *Instantiate Template* button.

When we performed the application build, there was no template. Rather, we selected the
builder image directly and OpenShift presented only the standard build workflow.
Now we are using a template - a preconfigured set of resources that includes
parameters that can be customized. In our case, the parameters we are concerned
with are -- user, password, database, and
admin password.

![MongoDB Deploy](/images/nationalparks-databases-catalog-databases-mongodb-config.png)

>CAUTION: Make sure you name your database service name *mongodb-nationalparks*

You can see that some of the fields say *"generated if empty"*. This is a
feature of *Templates* in OpenShift. For
now, be sure to use the following values in their respective fields:

* `Database Service Name` : `mongodb-nationalparks`
* `MongoDB Connection Username` : `mongodb`
* `MongoDB Connection Password` : `mongodb`
* `MongoDB Database Name`: `mongodb`
* `MongoDB Admin Password` : `mongodb`

>CAUTION: Make sure to have configured the *`MongoDB Database Name`* parameter with the appropriate value as by default it will already have a value of `sampledb`.

Once you have entered in the above information, click on *Create* to go to the next step which will allow us to add a binding.

From left-side menu, click to *Secrets*. Secrets are a mechanism to hold sensitive information such as passwords, configuration files, private source repository credentials, and so on. Secrets decouple sensitive content from the pods. 

![List Secrets](/images/nationalparks-databases-list-secrets.png)

Click the secret that begins *mongodb-ephemeral-parameters* like the one listed in the image above, that we will use for *Parameters*. The secret can be used in other components, such as the `nationalparks` backend, to authenticate to the database.

Now that the connection and authentication information stored in a secret in our project, we need to add it to the `nationalparks` backend. Click the *Add Secret to Workload* button.

![National Parks Binding](/images/nationalparks-databases-binding-view-secret.png)

Select the `nationalparks` workload and click *Save*.

![Add binding to application](/images/nationalparks-databases-binding-add-binding-to-nationalparks.png)

This change in configuration will trigger a new deployment of the `nationalparks` application with the environment variables properly injected.

Back in the *Topology* view, if required, whilst holding the shift key down, click and drag the `mongodb-nationalparks` component into the light gray area that denotes the `workshop` application, so that all three components are contained in it. (Note: Shift click and drag - this is how you move components in and out of the application boundary. You may also edit the *part-of* label for the component, for example all the components in this *workshop* application have this label *app.kubernetes.io/part-of=workshop* ).

![Add mongodb to the workshop app](/images/nationalparks-databases-add-mongodb-to-workshop-app.png)

Next, let's fix the labels assigned to the `mongodb-nationalparks` deployment. Currently, we cannot set labels when using the database template from the catalog, so we will fix these labels manually. 

Like before, we'll add 3 labels:

The name of the Application group:

```
app=workshop
```

Next the name of this deployment.

```
component=nationalparks
```

And finally, the role this component plays in the overall application.

```
role=database
```

Execute the following command:

```
oc label dc/mongodb-nationalparks svc/mongodb-nationalparks app=workshop component=nationalparks role=database --overwrite
```

## Exercise: Exploring OpenShift Magic

As soon as we attached the Secret to the *Deployment*, some
magic happened. OpenShift decided that this was a significant enough change to
warrant updating the internal version number of the *ReplicaSet*. You
can verify this by looking at the output of `oc get rs`:


```
NAME                       DESIRED   CURRENT   READY   AGE
nationalparks-58bd4758fc   0         0         0       4m58s
nationalparks-7445576cd9   0         0         0       6m42s
nationalparks-789c6bc4f4   1         1         1       41s
parksmap-57df75c46d        1         1         1       8m24s
parksmap-65c4f8b676        0         0         0       18m
```

We see that the DESIRED and CURRENT number of instances for the current deployment. The desired and current number of the other instances are 0.
This means that OpenShift has gracefully torn down our "old" application and
stood up a "new" instance.

## Exercise: Data, Data, Everywhere

Now that we have a database deployed, we can again visit the `nationalparks` web
service to query for data:

```
http://nationalparks-{{ project_namespace }}.{{cluster_subdomain}}/ws/data/all
```

And the result?

```
[]
```

Where's the data? Think about the process you went through. You deployed the
application and then deployed the database. Nothing actually loaded anything
*INTO* the database, though.

The application provides an endpoint to do just that:

```
http://nationalparks-{{ project_namespace }}.{{cluster_subdomain}}/ws/data/load
```

And the result?

```
Items inserted in database: 2893
```

If you then go back to `/ws/data/all` you will see tons of JSON data now.
That's great. Our parks map should finally work!

>NOTE: There are some errors reported with browsers like Firefox 54 that don't properly parse the resulting JSON. It's
a browser problem, and the application is working properly.

```
https://parksmap-{{ project_namespace }}.{{cluster_subdomain}}
```

Hmm... There's just one thing. The main map **STILL** isn't displaying the parks.
That's because the front end parks map only tries to talk to services that have
the right *Label*.

>You are probably wondering how the database connection magically started
>working? When deploying applications to OpenShift, it is always best to use
>environment variables, secrets, or configMaps to define connections to dependent systems.  This allows
>for application portability across different environments.  The source file that
>performs the connection as well as creates the database schema can be viewed
>here:
>
>```
>http://www.github.com/openshift-roadshow/nationalparks/blob/master/src/main/java/com/openshift/evg/roadshow/parks/db/MongoDBConnection.java#L44-l48
>```
>
>In short summary: By referring to bindings to connect to services
>(like databases), it can be trivial to promote applications throughout different
>lifecycle environments on OpenShift without having to modify application code.


## Exercise: Working With Labels

We explored how a *Label* is just a key=value pair earlier when looking at
*Services* and *Routes* and *Selectors*. In general, a *Label* is simply an
arbitrary key=value pair. It could be anything.

* `pizza=pepperoni`
* `pet=dog`
* `openshift=awesome`

In the case of the parks map, the application is actually querying the OpenShift
API and asking about the *Routes* and *Services* in the project. If any of them have a
*Label* that is `type=parksmap-backend`, the application knows to interrogate
the endpoints to look for map data.
You can see the code that does this
[here](https://github.com/openshift-roadshow/parksmap-web/blob/master/src/main/java/com/openshift/evg/roadshow/rest/RouteWatcher.java).


Fortunately, the command line provides a convenient way for us to manipulate
labels. `describe` the `nationalparks` service:

```
oc describe route nationalparks
```

```
Name:                   nationalparks
Namespace:              {{ project_namespace }}
Created:                2 hours ago
Labels:                 app=workshop
                        app.kubernetes.io/component=nationalparks
                        app.kubernetes.io/instance=nationalparks
                        app.kubernetes.io/name=nodejs
                        app.kubernetes.io/part-of=workshop
                        app.openshift.io/runtime=nodejs
                        app.openshift.io/runtime-version=10
                        component=nationalparks
                        role=backend  
Annotations:            openshift.io/host.generated=true                          
Requested Host:         nationalparks-{{ project_namespace }}.{{cluster_subdomain}}
                        exposed on router router 2 hours ago
Path:                   <none>
TLS Termination:        <none>
Insecure Policy:        <none>
Endpoint Port:          8080-tcp

Service:                nationalparks
Weight:                 100 (100%)
Endpoints:              10.1.9.8:8080
```

You see that it already has some labels. Now, use `oc label`:


```
oc label route nationalparks type=parksmap-backend
```

You will see something like:


```
route.route.openshift.io/nationalparks labeled
```

If you check your browser now:

```
https://parksmap-{{ project_namespace }}.{{cluster_subdomain}}/
```

![MongoDB](/images/nationalparks-databases-new-parks.png)

You'll notice that the parks suddenly are showing up. That's really cool!

