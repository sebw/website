## Exploring OpenShift's Logging Capabilities

OpenShift provides some convenient mechanisms for viewing application logs.
First and foremost is the ability to examine a *Pod*'s logs directly from the
web console or via the command line.

### Background: Container Logs

OpenShift is constructed in such a way that it expects containers to log all
information to `STDOUT`. In this way, both regular and error information is
captured via standardized Docker mechanisms. When exploring the *Pod*'s logs
directly, you are essentially going through the Docker daemon to access the
container's logs, through OpenShift's API.

>In some cases, applications may not have been designed to send all of their
>information to `STDOUT` and `STDERR`. In many cases, multiple local log files
>are used. While OpenShift cannot parse any information from these files, nothing
>prevents them from being created, either. In other cases, log information is
>sent to some external system. Here, too, OpenShift does not prohibit these
>behaviors. If you have an application that does not log to `STDOUT`, either because it
>already sends log information to some "external" system or because it writes
>various log information to various files, fear not.


## Exercise: Examining Logs

Since we already deployed our application, we can take some time to examine its
logs. In the Developer Perspective, from Topology view, click the `parksmap` entry and then the *Resources* tab. You should see a *View Logs* link next to the *Pod* entry.

![View Logs for Pod](/images/parksmap-view-logs-link.png)

Click the *View Logs* link and you should see a nice view of the *Pod*'s logs:

![Application Logs](/images/parksmap-logging-console-logs.png)

>WARNING: If you notice some errors in the log, that's okay. We'll remedy those in a little bit.

You also have the option of viewing logs from the command line. Get the name of
your *Pod*:


```
oc get pods
```

```
NAME               READY     STATUS    RESTARTS   AGE
parksmap-1-hx0kv   1/1       Running   0          5h
```

And then use the `logs` command to view this *Pod*'s logs:

```
oc logs parksmap-1-hx0kv
```

You will see all of the application logs scroll on your screen:

```
2019-05-22 19:37:01.433  INFO 1 --- [           main] o.s.m.s.b.SimpleBrokerMessageHandler     : Started.
2019-05-22 19:37:01.465  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2019-05-22 19:37:01.468  INFO 1 --- [           main] c.o.evg.roadshow.ParksMapApplication     : Started ParksMapApplication in 3.97 seconds (JVM running
 for 4.418)
2019-05-22 19:38:00.762  INFO 1 --- [MessageBroker-1] o.s.w.s.c.WebSocketMessageBrokerStats    : WebSocketSession[0 current WS(0)-HttpStream(0)-HttpPoll(
0), 0 total, 0 closed abnormally (0 connect failure, 0 send limit, 0 transport error)], stompSubProtocol[processed CONNECT(0)-CONNECTED(0)-DISCONNECT(0)]
, stompBrokerRelay[null], inboundChannel[pool size = 0, active threads = 0, queued tasks = 0, completed tasks = 0], outboundChannel[pool size = 0, active
 threads = 0, queued tasks = 0, completed tasks = 0], sockJsScheduler[pool size = 1, active threads = 1, queued tasks = 0, completed tasks = 0]
2019-05-22 19:44:11.517  INFO 1 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet 'dispatcherServlet'
2019-05-22 19:44:11.517  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization sta
rted
2019-05-22 19:44:11.533  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization com
pleted in 16 ms
2019-05-22 19:44:13.395  INFO 1 --- [nio-8080-exec-2] c.o.e.roadshow.rest.BackendsController   : Backends: getAll
```

>WARNING: If you scroll through the logs, you may notice an error that mentions a service account. What's that?  Never fear, we will cover that shortly.

--- 

## Application Health

### Background: Readiness and Liveness Probes
As we have seen before in the UI via warnings, there is a concept of application
health checks in OpenShift. These come in two flavors:

* Readiness probe
* Liveness probe

From the
https://{{DOCS_URL}}/applications/application-health.html[Application
Health] section of the documentation, we see the definitions:

- Liveness Probe::  
  A liveness probe checks if the container in which it is configured is still
  running. If the liveness probe fails, the kubelet kills the container, which
  will be subjected to its restart policy. Set a liveness check by configuring
  the `template.spec.containers.livenessprobe` stanza of a pod configuration.
- Readiness Probe::  
  A readiness probe determines if a container is ready to service requests. If
  the readiness probe fails a container, the endpoints controller ensures the
  container has its IP address removed from the endpoints of all services. A
  readiness probe can be used to signal to the endpoints controller that even
  though a container is running, it should not receive any traffic from a proxy.
  Set a readiness check by configuring the
  `template.spec.containers.readinessprobe` stanza of a pod configuration.

It sounds complicated, but it really isn't. We will use the web console to add
these probes to our `nationalparks` application.

## Exercise: Add Health Checks
As we are going to be implementing a realistic CI/CD pipeline, we will be doing
some testing of the "development" version of the application. However, in order
to test the app, it must be ready. This is where OpenShift's application health
features come in very handy.

We are going to add both a readiness and liveness probe to the existing
`nationalparks` deployment. This will ensure that OpenShift does not add any
instances to the service until they pass the readiness checks, and will ensure
that unhealthy instances are restarted (if they fail the liveness checks).

From the *Topology* view, click `nationalparks`. On the side panel, click the *Actions* dropdown menu and the select *Add Health Checks*.

![Add Health Checks](/images/nationalparks-application-health-menu.png)

Click to *Add Readiness Probe* and add in *Path* field: 

```
/ws/healthz/
```

Leave all default settings like *Port* 8080 and *Type* HTTP GET. Click the little bottom-right confirmation gray tick to confirm:

![Add Readiness and Liveness Probe](/images/nationalparks-application-health-settings.png)

Repeat the same procedure for Liveness Probe, click to *Add Liveness Probe* and add in *Path* field: 

```
/ws/healthz/
```

Leave all default settings like *Port* 8080 and *Type* HTTP GET. Click the little bottom-right confirmation gray tick to confirm.

>WARNING: Note the trailing slash in the URL

Finally confirm all new changes clicking to *Add*:

![Add Probes](/images/nationalparks-application-health-add.png)

You will notice that these changes caused a new deployment -- they counted as a
configuration change.