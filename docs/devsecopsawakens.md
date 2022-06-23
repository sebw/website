## Remote Access to Your Application

Containers are treated as immutable infrastructure and therefore it is generally
not recommended to modify the content of a container through SSH or running custom
commands inside the container. Nevertheless, in some use-cases, such as debugging
an application, it might be beneficial to get into a container and inspect the
application.

## Exercise: Remote Shell Session to a Container Using the CLI

OpenShift allows establishing remote shell sessions to a container without the
need to run an SSH service inside each container. In order to establish an
interactive session inside a container, you can use the `oc rsh` command. First
get the list of available pods:

``` 
oc get pods
``` 

You should an output similar to the following:

``` 
NAME                        READY   STATUS    RESTARTS   AGE
parksmap-65c4f8b676-fxcrq   1/1     Running   0          52m
``` 

Now you can establish a remote shell session into the pod by using the pod name:


``` 
oc rsh parksmap-65c4f8b676-fxcrq
``` 

You would see the following output:


``` 
sh-4.2$
``` 

>The default shell used by `oc rsh` is `/bin/sh`. If the deployed container does
>not have *sh* installed and uses another shell, (e.g. *A Shell*) the shell command
>can be specified after the pod name in the issued command.

Run the following command to list the files in the top folder:

``` 
ls /
``` 


``` 
anaconda-post.log  bin  dev  etc  home  lib  lib64  lostfound  media  mnt  opt  parksmap.jar  proc  root  run  sbin  srv  sys  tmp  usr  var
``` 

## Exercise: Remote Shell Session to a Container Using the Web Console

The OpenShift Web Console also provides a convenient way to access a terminal session on the container without having to use the CLI.

In order to access a pod's terminal via the Web Console, go to the Topology view in the Developer Perspective, click the `parksmap` entry, and then click on the *Pod*. 

![Pod in Dev Console](/images/parksmap-rsh-dev-console-pod.png)

Once you are viewing the information for the selected pod, click on the *Terminal* tab to open up a shell session.

![Pod List](/images/parksmap-rsh-applications-pods-terminal.png)

Go ahead and execute the same commands you did when using the CLI to see how the Web Console based terminal behaves.

Before proceeding, close the connection to the pod.

``` 
exit
``` 

## Exercise: Execute a Command in a Container

In addition to remote shell, it is also possible to run a command remotely in an
already running container using the `oc exec` command. This does not require
that a shell is installed, but only that the desired command is present and in
the executable path.

In order to show just the JAR file, run the following:

``` 
oc exec parksmap-2-mcjsw -- ls -l /parksmap.jar
``` 

You would see something like the following:


``` 
-rw-r--r--. 1 root root 39138901 Apr  1 16:54 /parksmap.jar
``` 

>The `--` syntax in the `oc exec` command delineates where exec's options
>end and where the actual command to execute begins. Take a look at `oc exec
>--help` for more details.

You can also specify the shell commands to run directly with the *oc rsh* command:

``` 
oc rsh parksmap-2-mcjsw whoami
``` 

You would see something like:

``` 
1000580000
``` 

>It is important to understand that, for security reasons, OpenShift does not run containers as the user specified in the Dockerfile by default. In fact,
>when OpenShift launches a container its user is actually randomized.

>If you want or need to allow OpenShift users to deploy container images that do
>expect to run as root (or any specific user), a small configuration change is
>needed. You can learn more about the
>[container image guidelines](https://{{DOCS_URL}}/openshift_images/create-images.html#images-create-guide-general_create-images)
>for OpenShift.




## Role-Based Access Control

Almost every interaction with an OpenShift environment that you can think of
requires going through the OpenShift's control plane API. All API interactions are both authenticated (AuthN - who are you?) and authorized (AuthZ - are you allowed to do what you are asking?).

## Exercise: Apply RBAC

In the log aggregation lab we saw that there was an
error in reference to a *Service Account*.

As OpenShift is a declarative platform, some actions will be performed by the platform and not by the end user (when he or she issues a command). These actions are performed using a *Service Account* which is a special type of `user` that the platform will use internally.

OpenShift automatically creates a few special service accounts in every project.
The **default** service account is the one taking the responsibility of running the pods, and OpenShift uses and injects this service account into
every pod that is launched. By changing the permissions for that service
account, we can do interesting things.

You can view current permissions in the web console, go to the Topology view in the Developer Perspective, click the `parksmap` entry, go to the *Details* tab, and then click the *Namespace*. 

![Namespace](/images/parksmap-permissions-namespace.png)

Then, click *Role Bindings*.

![Membership](/images/parksmap-permissions-membership.png)

== Exercise: Grant Service Account View Permissions
The parksmap application wants to talk to the OpenShift API to learn about other
*Pods*, *Services*, and resources within the *Project*. You'll soon learn why!

``` 
oc project {{ project_namespace }}
``` 

Then:

``` 
oc policy add-role-to-user view -z default
``` 

The `oc policy` command above is giving a defined _role_ (`view`) to a user. But
we are using a special flag, `-z`. What does this flag do? From the `-h` output:

``` 
-z, --serviceaccount=: service account in the current namespace to use as a user
``` 

The `-z` syntax is a special one that saves us from having to type out the
entire string, which, in this case, is
`system:serviceaccount:{{ project_namespace }}:default`. It's a nifty shortcut.

>The `-z` flag will only work for service accounts in the *current* project.
>If you're referring to a service account in a different project, use the `-n <project>`switch.

Now that the `default` *Service Account* now has **view** access, so now it can query the API to see what resources are within the *Project*. This also has the added benefit of suppressing the error message! Although, in reality, we fixed the application.

Another way you could have done the same is by using the OpenShift console. Once you're on the 
*Workloads -> Deployments* page, click on the *Namespace*, then *Role Bindings* and then the *Create Binding* button.

![Service account list](/images/parksmap-permissions-membership-serviceaccount-list.png)

Select *view* for the Role Binding Name *{{ project_namespace }}* for the Namespace, *view* for the Role Name, *Service Account* for the Subject, *{{ project_namespace }}* for the Subject Namespace, and *default* for the Subject Name.

![Service account edit](/images/parksmap-permissions-membership-serviceaccount-edit.png)

Once you're finished editing permissions, click on the *Create* button.

![Service account changed](/images/parksmap-permissions-membership-serviceaccount-done.png)

## Exercise: Redeploy Application
One more step is required. We need to re-deploy the `parksmap` application because it's
given up trying to query the API.

``` 
oc rollout restart deployment/parksmap
``` 

A new deployment is immediately started. Return to Topology view and click the `parksmap` entry again to watch it happen. You might not be fast enough! But it will be reflected in the *ReplicaSet* number.

![Application deployed](/images/parksmap-permissions-redeployed.png)

If you look at the logs for the application now, you should see no errors.  That's great.

## (Optional) Exercise: Grant User View Permissions
If you create a project, you are that project's administrator. This means that
you can grant access to other users, too. If you like, give your neighbor view
access to your project using the following command:

CAUTION: In the following command(s), replace `{{ project_namespace  }}` with the user name of the person to whom you want to grant access.


``` 
oc policy add-role-to-user view {{ project_namespace  }}
``` 

Have them go to the project view by clicking the *Projects* button and verify
that they can see your project and its resources. This type of arrangement (view
but not edit) might be ideal for a developer getting visibility into a
production application's project.




--- 

## Implementing network isolation between containers

The goal of this lab is to learn about how to implement network isolation between running containers in Red Hat OpenShift Container Platform using Software Defined Networking and Network Policies. First, we will create a few projects (K8s namespaces) and examine default out of the box network policies provided in OpenShift. Then, we will use Network Policies to restrict which appications/projects can talk to each other by restricting the network layer to provide that network isolation between running containers with Software Defined Networking and Network Policies.

### Introduction

[Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/ ) are an easy way for Project Administrators to define exactly what ingress/egress traffic is allowed to any pod, from any other pod, including traffic from pods located in other projects. By default, all Pods in a project are accessible from all other Pods and network endpoints. To isolate one or more Pods in a project, you can create NetworkPolicy objects in that project to indicate the allowed incoming connections. Project administrators can create and delete NetworkPolicy objects within their own project.

If a Pod is matched by selectors in one or more NetworkPolicy objects, then it will accept only connections that are allowed by at least one of those NetworkPolicy objects. A Pod that is not selected by any NetworkPolicy objects is fully accessible.

### Background
The objective of the exercise below is to create multiple projects and to test application communication between them. You will introduce Network policies that will block application communications across projects and then allow some.

Project-c will have a hello-world application to which you will send requests.
Projects-a, b and c will all have a client application. The client from project-c will always be able to communication with hello-world in project-c since they are in the same project. However, depending on the network policies in place at times you will be blocked from communicating from clients in projects a and b to the hello-world application in project c.

## Exercise: Creating Projects and Labeling Namespaces

As a cluster admin user, create 3 projects and label those namespaces.

```
[localhost ~]$ oc new-project proj-a --display-name="Project A"
[localhost ~]$ oc new-project proj-b --display-name="Project B"
[localhost ~]$ oc new-project proj-c --display-name="Project C"
```

In order to create Network Policies and allow applications in one namespace to be accesssed by applications running in only certain other namespace, you have to label the namespaces so they can be identified in Network Policies (and this is why you have to be 'cluster-admin' level user):

```
oc label namespace proj-a name=proj-a
oc label namespace proj-b name=proj-b
oc label namespace proj-c name=proj-c
```

Now, let's look at the projects and labels we just created:

```
oc get projects proj-a proj-b proj-c --show-labels 
```

![](/images/lab2.1-showlabels.png)

## Exercise: Creating the 'hello world' microservice and client pod in proj-c

Let's go into the project named *proj-c* and create 2 pods and a service.


```
[localhost ~]$ oc project proj-c
[localhost ~]$ oc new-app quay.io/bkozdemb/hello --name="hello"
```
This will create a new app, which is a 'hello world' container based on image stored in quay.io that’s built on the RHEL base python image. It runs a simple web server that prints 'hello'.

Next, let's confirm that the 2 pods are starting to run.


```
oc get pods
```

![](/images/lab2.1-ocgetpods.png)

Now, let's create and run the client pod, which is a Fedora base image. When the image is running, notice the command that’s being run (tail -f /dev/null), which essentially prevents the pod from running and then immediately quitting. This client pod will be used to run a curl command later.


```

[localhost ~]$ oc run client --image=fedora --command -- tail -f /dev/null
pod/client created
```

Let's confirm that our client pod and hello world pod are now running.


```
oc get pods
```


![](/images/lab2.1-ocgetpods2.png)

Next, let's create similar client pods in other projects named *proj-a* and *proj-b*.


```

[localhost ~]$ oc project proj-b
[localhost ~]$ oc run client  --image=fedora --command -- tail -f /dev/null
[localhost ~]$ oc project proj-a
[localhost ~]$ oc run client  --image=fedora --command -- tail -f /dev/null

```

Notice that the projects , *proj-a* and *proj-b* just have client pods running.


```
oc get pods -n proj-a
oc get pods -n proj-b
```

![](/images/lab2.1-ocgetpods3.png)

As we saw in the previous steps, *proj-c*, has both a client pod and the service (as a part of 'hello world' app).


```
oc get pods -n proj-c
```

![](/images/lab2.1-ocgetpods4.png)

When the client pod is ready, display pod information with their labels.


```
oc get pods --show-labels
```

![](/images/lab2.1-showlabels2.png)


Notice that the label that we’re using for Client pod is *run=client* which is created automatically by our previous 'oc run client' command.

Next, from a different project, *proj-a*, connect to the Client container and try to access the *hello.proj-c* service in project, *proj-c*. The default network policy allows a client pod in *proj-a* to access the microservice in *proj-c*.

```
oc project proj-a
POD=`oc get pods --selector=run=client --output=custom-columns=NAME:.metadata.name --no-headers`
```
This command above simply assigns the pod name to the *POD* variable since some pods have random names by default so this command allows you to give a specific name to the pod.

```
echo $POD
```

![](/images/lab2.1-echopod.png)

This returns *client*, which is the pod name.
Next, go into the pod and curl the service in project, *proj-c*. Notice that this is allowed since it's open access by default.

```
oc rsh ${POD}
#Inside the pod you have to execute:
curl -v hello.proj-c:8080
```

![](/images/lab2.1-curloutput1.png)

What you have seen so far is how Network Policies work by default in OpenShift. 
Now let's take a look at the default Network Policies in the OpenShift web console. URL of web console can be found by running command:

```
[localhost ~]$ oc whoami --show-console
https://console-openshift-console.apps.cluster-tx8sn.tx8sn.sandbox1590.opentlc.com
```

Log into the OpenShift web console, then go to Projects and find the project, *proj-c*. Navigate into *proj-c*, then select *Networking* -> *Network Policies*.

![](/images/lab2.1.10-webconsole2.png)
![](/images/lab2.1.10-webconsole1.png)

Notice that (in earlier versions of OpenShift) those two Network Policies are created by default:

* *allow-from-all-namespaces*: This is why we can hit services in the project, *proj-c* from other projects (such as projects, *proj-a* and *proj-b*).
* *allow-from-ingress-namespace*: This allows ingress from the router (outside in through the router).

>NOTE: In the recent versions of OpenShift 4.x those default Network Policies are no longer present. As a result, if no Network Policies are defined, all traffic is allowed.

## Exercise: Creating Network Policies for network isolation
In the OpenShift web console, choose project, *proj-c*, and go to *Networking* -> *Network Policies*.

Next, delete the 2 default Network Policies (*allow-from-all-namespaces* and *allow-from-ingress-namespace*) if you see them. Remember that if no Network Policies are defined, all traffic is allowed.

![](/images/lab2.2.2-deletenetworkpolicies.png)

Now, create a new Network Policy in project, *proj-c* that denies traffic from other namespaces. It should be
the first example shown on the right in the Sample Network policies. Notice there are a lot of Sample Network Policies. Apply the first example *Limit access to the current namespace*. Click Try it. This creates the yaml. Next, press *create*.

![](/images/lab2.2-createnetworkpolicies1.png)
![](/images/lab2.2-createnetworkpolicies2-new.png)


Now, navigate into *Networking* -> *Network Policies*. and notice that the *deny-other-namespaces* network policy is defined.

![](/images/lab2.2-denyothernamespaces.png)

Next, try to curl the hello world service in project, *proj-c* from the client in *proj-a*. Notice that the curl fails this time.


```
oc rsh ${POD}
#Inside the pod you have to execute:
curl -v hello.proj-c:8080
```

![](/images/lab2.2-curlfail.png)

Remember to exit the pod with the `exit` command.

Same kind of failure you would get if you try to access application running in *proj-c* from *proj-b* because the *deny-other-namespaces* Network Policy blocks traffic from ALL namespaces

## Exercise: Creating Network Policies for selective network access

Here you will create additional Network Policy that will allow access to pods running in *proj-c* project from those running in different projects, selected by their labels. In the previous lab you created a Network Policy that denies access to all pods in *proj-c* from other projects.   
Now, let's create a Network Policy that is based on the sample "ALLOW traffic from all Pods in a particular namespace" policy. In the 'podSelector.matchLabels' section specify 'deployment:hello' to select the 'hello' labeled pods and in the 'namespaceSelector.matchLabels' section specify 'name:proj-a' to indicate that you will allow traffic from apps deployed in that namespace (recall that we previously labeled it with 'name:proj-a'). Press *Create* to create Network Policy

![](/images/lab2.3-allow-traffic-from-proj-a.png)

Now, navigate into *Networking* -> *Network Policies*. and notice that the *web-allow-production* Network Policy is there:

![](/images/lab2.3-policies-list.png)

Next, again try to access the 'hello world' service in project *proj-c* from the Client running in *proj-a*. Notice that the curl succeeds this time because ingress traffic is explicitely allowed from *proj-a* to our 'hello world' pod by the *web-allow-production* Network Policy:


```
[localhost ~]$ oc rsh ${POD}
sh-5.0# curl -v hello.proj-c:8080
```

![](/images/lab2.3-curl-from-proj-a-ok.png)

Next, try to curl the 'hello world' service in project *proj-c* from the Client running in *proj-b*. Notice that the curl fails because the first Network Policy still blocks it and the second one is not applicable to pods running in *proj-b*:


```
[localhost ~]$ oc rsh ${POD}
sh-5.0# curl -v hello.proj-c:8080
```

![](/images/lab2.3-curl-from-proj-b-fails.png)

### Background

You have learned how to created multiple OpenShift projects/namespaces and test application communication between them. You also learned how to create declarative Network Policies that block application communications across projects and then allow application communications between selected applications running in specific namespaces. Network Policies when used propely are very powerful way to implement cloud native applications network security.
