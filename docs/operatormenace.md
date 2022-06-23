
>In this section, the learners will split in different teams to understand the dynamics between the different teams when it comes to accomplish a simple task like deploying an application on a virtual machine.

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

>WARNING: Don't forget to scale down back to 1 instance your `parksmap` component as otherwise you might experience some weird behavior in later labs. This is due to how the application has been coded and not to OpenShift itself.