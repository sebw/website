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
anaconda-post.log  bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  parksmap.jar  proc  root  run  sbin  srv  sys  tmp  usr  var
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
-z, --serviceaccount=[]: service account in the current namespace to use as a user
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