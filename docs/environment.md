
## Architecture Overview 
Never forget that a drawings is better than a thousands words! Even if your drawing skills did not evolve since kindergarden :wink:

This lab introduces you to the architecture of the ParksMap application used throughout this workshop, to get a better understanding of the things you'll be doing from a developer perspective.  
ParksMap is a polyglot geo-spatial data visualization application built using the microservices architecture and is composed of a set of services which are developed using different programming languages and frameworks.

![application](/images/roadshow-app-architecture.png)

The main service is a web application which has a server-side component in charge of aggregating the geo-spatial APIs provided by multiple independent backend services and a client-side component in JavaScript that is responsible for visualizing the geo-spatial data on the map. The client-side component which runs in your browser communicates with the server-side via WebSockets protocol in order to update the map in real-time.

There will be a set of independent backend services deployed that will provide different mapping and geo-spatial information. The set of available backend services that provide geo-spatial information are:

* WorldWide National Parks
* Major League Baseball Stadiums in North America

The original source code for this application is located [here](https://github.com/openshift-roadshow/).

The server-side component of the ParksMap web application acts as a communication gateway to all the available backends.   
These backends will be dynamically discovered by using service discovery mechanisms provided by OpenShift which will be discussed in more details in the following labs.

## Exploring the Environment
###  Command Line Interface

OpenShift includes a feature-rich web console with both an Administrator perspective and a Developer perspective. In addition to the web console, OpenShift includes command line tools
to provide users with a nice interface to work with applications deployed to the
platform.  The `oc` command line tool is an executable written in the Go
programming language and is available for the following operating systems:

- Microsoft Windows
- macOS 10
- Linux

This lab environment requires the `oc` command line tool to be installed. [Download here.](https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/latest/).

Issue the following command to see help information:

```
oc help
```

### Using a Project

Projects are a top level concept to help you organize your deployments. An
OpenShift project allows a community of users (or a user) to organize and manage
their content in isolation from other communities. Each project has its own
resources, policies (who can or cannot perform actions), and constraints (quotas
and limits on resources, etc). Projects act as a "wrapper" around all the
application services and endpoints you (or your teams) are using for your work.

During this lab, we are going to use a few different commands to make sure that
things in the environment are working as expected.  Don't worry if you don't
understand all of the terminology as we will cover it in detail in later labs.

In this lab environment, you already have access to single project: *{{ project_namespace  }}*.

If you had multiple projects, the first thing you would want to do is to switch
to the *{{ project_namespace  }}* project to make sure you're on the correct project from now on.
You can do this with the following command:

```
oc project {{ project_namespace  }}
```

### The Web Console

OpenShift ships with a web-based console that will allow users to
perform various tasks via a browser. 

To get a feel for how the web console works, click on this http://console-openshift-console.{{cluster_subdomain}}/k8s/cluster/projects[Web Console] link.

On the login screen, enter the following credentials:

```
Username: username
Password: openshift
```
The first time you access the web console, you will most likely be in the Administrator perspective. You will be presented with the list of Projects that you can access, and you will see something that looks like the following image:

![Web Console](/images/explore-webconsole1sc.png)

Click on the *{{ project_namespace  }}* project link. When you click on the
*{{ project_namespace  }}* project, you will be taken to the project details page,
which will list some metrics and details about your project. There's nothing there now, but that will change as you progress through the lab.

![Explore Project](/images/explore-webconsole2.png)

At the top of the left navigation menu, you can toggle between the Administrator perspective and the Developer perspective.

![Toggle Between Perspectives](/images/explore-perspective-toggle.png)

Select *Developer* to switch to the Developer perspective. Once the Developer perspective loads, you should be in the *Topology* view. Right now, there are no applications or components to view, but once you begin working on the lab, you'll be able to visualize and interact with the components in your application here.

![Add New Applications](/images/explore-topology-view.png)

We will be using a mix of command line tooling and the web console for the labs.
Get ready!



--- 

openshift_api_url: https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443
openshift_client_download_url: http://d3s3zqyaz8cp2d.cloudfront.net/pub/openshift-v4/clients/ocp/stable-4.8/openshift-client-linux.tar.gz
openshift_console_url: https://console-openshift-console.apps.cluster-bz2cd.bz2cd.sandbox565.opentlc.com
users:
  user1: {login_command: 'oc login -u user1 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user10: {login_command: 'oc login -u user10 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user11: {login_command: 'oc login -u user11 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user12: {login_command: 'oc login -u user12 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user13: {login_command: 'oc login -u user13 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user14: {login_command: 'oc login -u user14 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user15: {login_command: 'oc login -u user15 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user16: {login_command: 'oc login -u user16 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user17: {login_command: 'oc login -u user17 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user18: {login_command: 'oc login -u user18 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user19: {login_command: 'oc login -u user19 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user2: {login_command: 'oc login -u user2 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user20: {login_command: 'oc login -u user20 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user21: {login_command: 'oc login -u user21 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user22: {login_command: 'oc login -u user22 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user23: {login_command: 'oc login -u user23 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user24: {login_command: 'oc login -u user24 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user25: {login_command: 'oc login -u user25 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user26: {login_command: 'oc login -u user26 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user27: {login_command: 'oc login -u user27 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user28: {login_command: 'oc login -u user28 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user29: {login_command: 'oc login -u user29 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user3: {login_command: 'oc login -u user3 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user30: {login_command: 'oc login -u user30 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user4: {login_command: 'oc login -u user4 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user5: {login_command: 'oc login -u user5 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user6: {login_command: 'oc login -u user6 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user7: {login_command: 'oc login -u user7 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user8: {login_command: 'oc login -u user8 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}
  user9: {login_command: 'oc login -u user9 -p openshift https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443',
    password: openshift}

Openshift Console: https://console-openshift-console.apps.cluster-bz2cd.bz2cd.sandbox565.opentlc.com
Openshift API for command line 'oc' client: https://api.cluster-bz2cd.bz2cd.sandbox565.opentlc.com:6443
Download oc client from http://d3s3zqyaz8cp2d.cloudfront.net/pub/openshift-v4/clients/ocp/stable-4.8/openshift-client-linux.tar.gz

HTPasswd Authentication is enabled on this cluster.
Users user1 .. user30 are created with password `openshift`
User `opentlc-mgr` with password `r3dh4t1!` is cluster admin.
Nexus password is admin123

Getting Started workshop provisioned for 30 user(s)

Lab access (Homeroom Route) URL to give to students:
https://homeroom-labs.apps.cluster-bz2cd.bz2cd.sandbox565.opentlc.com
