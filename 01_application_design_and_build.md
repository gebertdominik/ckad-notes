# Application design and build (20%)

## Objectives

* Define, build and modify container images
* Understand Jobs and CronJobs
* Understand multi-container Pod design patterns(e.g. sidecar, init and others)
* Utilize persistent and ephemeral volume


# Build

## Objectives

* Learn about runtime and container options
* Contenerize an application
* Host a local repository
* Deploy a multi-container Pod
* Configure liveness and readiness Probess

## Container Options

K8s is being developed to work with many container engines. Docker usually is the first choice, but as other container engines become mature, k8s become more open and independent. Docker Engine remains the default for Kubernetess, through CRI-O and others are gaining popularity.

The contenerized image is moving from Docker to one that is not bound to higher-level tools and that is more portable across operating systems and environments. **The Open Container Initiative (OCI)** was formed to help with this. Docker donated their `libcontainer` projec to form a new codebase called `runC` to support those goals.

> **Note**
> 
> More information about `runC` can be found on [GitHub](https://github.com/opencontainers/runc)

Where Docker was once the only real choice for developers, the trend toward open specifications and flexibility indicates taht building with vendor-neutral features is a wise choice.

### Container Runtime Interface (CRI)

The goal of CRI is to allow easy integration of container runteimes with kubelet. By providing a protobuf method for API, specs and libraries, new runtimes can easily be integrated without needing deep understanding of kubelet internals.

### CRI-O

This project is currently in incubation as part of Kubernetes. It uses the Kubernetes Container Runtime Interface with OCI-compatible runtimes. Currently, there is support for `runC` (default) and `Clear Containers`, but a stated goal of the project is to work with any OCI-compliant runtime.

While newer than Docker or rkt, this project has gained major vendor support due to its flexibility and compatibility.

### containerd

The `containerd` project is focused on exposing highly-decoupled low-level primitives:

* Defaults to `runC` to run containers according to the OCI Specifications
* Intended to be embedded into larger systems
* Minimal CLI, focused on debugging and development

This project is better suited to integration and operation teams building specialized products, instead of typical build, ship and run application.

## Contenerizing an Application

Not all applications do well with contenerization. The more stateless and transient the application, the better. Also, any environmental configuration should be removed, as those can be provided using other tools, like ConfigMaps and secrets. The app should be developed until it can be deployed to multiple environments without needing to be changed. Many legacy applications become a series of objects and atifacts, residing among multiple containers.

The use of Docker had been the industry standard. Large companies like Red Hat are moving to other open source tools. While currently new, we can expect them to become the new standard in the future. 

> Those new tools are for example:
> 
> * Buildah
> 
> It's a tool that focuses on creating Open Container Initiative (OCI) images. It allows for creating images with and without Dockerfile.
> 
> * Podman
> 
> A "Pod manager" tool, allows for the life cycle management of a container. We could consider it as a replacement for docker. Red Hat's goal was to replace all Docker functionallity with podman, using the same syntax to minimize transition issues. Some places alias docker to actually run podman.

Contenerizing legacy(monolithic) aplication often brings a question if the application should be contenerized as it is, or rewritten as transient, decoupled microservice. The cost of rewriting legacy applications can be hight, but there is also value to leveraging the flexibility of k8s.

## Creating dockerised app

`docker build` process is responsible for pulling everything in the app directory when it creates the image. It searches for file with name `Dockerfile`. Steps to create container with the app are as follows:

* Create `Dockerfile`
* Build the container `sudo docker build -t simpleapp`
* Verify the image `sudo docker images`
* `sudo docker run simpleapp`
* Push to the repository `sudo docker push`

While Docker has made it easy to upload images to their Hub, these images are then public and accessible to the world. A common alternative is to create a local repository and push images there.

Once we can push and pull images, we can run a new deployment using the image. T

`kubectl create deployment <Deploy-Name> --image=<repo>/<app-name>:<version>`

Usually we can run commands inside a container by executing

`kubectl exec -it <Pod-Name> -- /bin/bash`

## Multi-Container Pod

It may not make sense to create an entire image to add functionality like a shell or logging agent. Instead, we can add another container to the Pod, which would provide the necessary tools.

Each container in the Pod should be transient and decoupled. If adding another container limits the scalability or transient nature of the original app, then a new build may be warranted.

Every container in a Pod shares a single IP address and namespace. Each container has equal potential access to storage given to the Pod. K8s does not provide any locking, so our configuration or app should be such that containers do not have conflicting writes.

There are three terms often used for multi container pods: `ambassador`, `adapter`, and `sidecar`. Based off the documentation, we may think there is some setting for each. There is not. Each term is an expression of what a secondary pod is intended to do. All are just multi-container pods.

> **Ambassador**
> This type of secondary container would be used to communicate with outside resources, often otside the cluster. Using a proxy, like Envoy or other, we can embed a proxy instead of using one provided by the cluster. It is helpful if you are unsure of the cluster configuration.

> **Adapter**
> This type of secondary container is useful to modify the data generated by the primary container. For example, the Microsoft version of ASCII is distinct from everyone else. We may need to modify a datastream for proper use.

> **Sidecar**
> Similar to sidecar on a motorcycle, it does not provide the main power, but it does help carry stuff. A sidecar is a secondary container which helps or provides a service not found in the primary application. Logginc containers are a common sidecar.

## Readiness/Liveness/Startup Probes

* readinessProbe

Rather than communicate wit h a client prior to being fully ready, we can use a `readinessProbe`. The container will not accept traffic until the probe returns a healthy state.

With the `exec` statement, the container is not considered ready until a command returns a zero exit code. As long as the return is non-zero, the container is considered not ready and the probe will keep trying.

Another type of probe uses an HTTP GET request(`httpGet`). The container is not considered healthy until the web server returns a code **200-399**. Any other code indicates failuter, and the probe will try again.

The TCP Socket probe (`tcpSocket`) will attempt to open a socket on a predetermined port, and keep trying based on periodSeconds. Once the port can be opened, the container is considered healthy.

* livenessProbe

Just as we want to wait for a container to be ready for traffic, we also want to make sure it stays in a healthy state. Some applications may have built-in checking, so we can use `livenessProbe` to continually check the health of a container. If the container is found to fail a probe, it is terminated. If under a controller, a replacement would be spawned.

* startupProbe

Useful for testing an app which takes a long time to start. If kubelet uses `startupProbe`, it will disable liveness and readiness checks until the app passes the test. The duration until the container is considered failed is failureThreshold times periodSeconds. For example if our periodSeconds was set to five seconds, and our failureThreshold was set to ten, kubelet would check the app every five seconds until it succeeds, or is considered failed after a total of 50 seconds.

## Testing

While custom-built tools may be best at testing a deployment, there are some built-in `kubectl` arguments to begin the process. The first one is `describe` and the next would be `logs`

Details, conditions, volumes and events for an object can be seen with describe. The `Events` at the end of the output presents a chronological and node view of cluster actions and associated messages. A simple nginx pod may show the following output:

`kubectl describe pod test 1`

```
....
Events:  
  Type    Reason     Age   From               Message  
  ----    ------     ----  ----               -------  
  Normal  Scheduled  18s   default-scheduler  Successfully assigned default/test1 to master   
  Normal  Pulling    17s   kubelet, master    Pulling image "nginx"  
  Normal  Pulled     16s   kubelet, master    Successfully pulled image "nginx"  
  Normal  Created    16s   kubelet, master    Created container nginx  
  Normal  Started    16s   kubelet, master    Started container nginx
```

A next step in testing may be to look at the output of containers within a pod. Not all applications will generate logs, so it could be difficult to know if the lack of output is due to error or configuration.

`kubectl logs test1`

## Helm

In addition to pushing the image to a repository, we may want to provide the image and other objects, such that they can be easily deployed. The Helm package manager is the package manager for K8s.

Helm uses a chart, or collection of YAML files to deploy one or more objects. For flexibility and dynamic installation, a `values.yaml` file is a part of a chart and is often edited before deployment.

A chart can come from many locations with [ArtifactHub](https://artifacthub.io/) becoming a centralized site to publish and find charts.


# Design

## Objectives

* Define an application's resource requirements.
* Understand multi-container Pod design patterns: sidecar, adapter, ambassador.
* Discuss application design concepts.

## Traditional Applications - Considerations

Optimal k8s deployment design changes are more than just contenerisation of an application. Traditional apps were built and deployed with the expectation of long-term processes and strong interdependence.

For example, and Apache web server is highly customizable. Often, the server would be configured and tweaked without interruption. The app may be migrated to larger servers if needed. The build and maintenance of the app assumes the instance would run without reset and have persistent and tightly coupled connections to other resources, such as networks and storage.

In early usage of containers, apps were contenerized without redevelop,ent. This lead to issues after resource failure, or upon upgrade, or configuration. The cost and hassle of redesign and re-implementation should be taken into account.

## Decoupled Resources

The use of decoupled resources is integral to k8s. Instead of an application using a dedicated port and socket, for the life of the instance, the goal is for each component to be decoupled from other resources. The expectation and software development toward separation allows for each component to be removed, replaced, or rebuilt.

Instead of hard-coding a resource in an application, an intermediary, such as Service, enables connection and reconnection to other resources, providing flexibility. A single machine is no longer required to meed the application and user needs - any number of systems coyld be brought together to meet the those needs when, and, as long as, necessary.

Also Kubernetes devleopers can optimize a particular function with fewer considerations of others objects.

## Transience

Equally important is the expectation of transience. Each object should be developed with the expectation that other componenets will die and be rebuilt. With any and all resources planned for transient relationships to others, we can update versions or scale usage in an easy manner.

An upgrade is perhaps not quite correct therm, as the existing app doeas not survive. Instead controller terminates the container and deploys a new one to replace it, using a different version of the application or setting. Typically, traditional apps were not written this way, opting toward long-term relationships for efficiency and ease of use.

## Flexible Framework

Resources working together, but decoupled from each other and without expectation of individual permanent relationship, gain flexibility, higher availability and easy scalability. Instead of a monolithic Apache server, we can deploy a flexible number of nginx servers, each handling a small part of the workload. The goal is the same, but the framework of the solution is distinct.

A decoupled, flexible and transient application and framework is not the most efficient. In order for the k8s orhestration to work, we need a series of agents, otherwise known as controllers or watch-loops, to constantly monitor the current cluster state and make changes until that state matches the declared configuration.

The commoditization of hardware has enabled the use of many smaller servers to handle a larger workload, instead of single, huge systems.

## Managing Resource Usage

K8s allows us to scale clusters. An understanding of how the k8s clusters view the resources is an important consideration. The `kube-scheduler`, or a custom scheduler, uses the `PodSpec` to determine the best node for deployment.

In addition to administrative tasks to grow or shrink the cluster or the number of Pods, there are autoscalers which can add or remove nodes or pods, with plans for one which uses `cgroup` settings to limit CPU and memory usage by individual containers.

By default, Pods use as much CPU and memory as the workload requires, behaving and coexisting with other Linux processes. Through the use of resource requests, the scheduler will only schedul a Pod to a node if resources exist to meet all requests on that node. The scheduler takes these and several other factors into account when selecting a node for use.

Monitoring the resource usage cluster-wide is not an included feature of k8s. Other projects, like Prometheus, are used instead. In order to view resource consumption issues locally, we can use `kubectl describe pod` command. We may only know of issues after the pod has been terminated.

### CPU

CPU requests are made in CPU units - millicores(or millicpu). A request for .7 of a CPU would be 700 millicore. Should a container use more resources than allowed, it won't be killed, but will be throttled. If the limits are set to the pod instead of a particular container, all usage of the containers is considered and all containers are throttled at the same time.

Each millicore is not evaluated, so using more than the limit is possible. The exact amount of overuse is not definite. Note the notation or syntax often found in the documentation:

```
spec.containers[].resources.limits.cpu
spec.containers[].resources.requests.cpu
```

The value of CPUs is not relative. It does not matter how many exists, or if other Pods have requirements. One CPU in k8s is equivalent to:

* 1 AWS vCPU
* 1 GCP Core
* 1 Azure vCore
* 1 Hyperthread on a bare-metal processor with Hyperthreading.

### Memory(RAM)

With Docker entinge, the `limits.memory` value is converted to an integer value and becomes the value to the `docker run --memory <value> <image>` command. The handling of a container which exceeds its memory limit is not definite. It may be restarted, or, if it asks for more than the memory request setting, the entire Pod may be evicted from the node.

```
spec.containers[].resources.limits.memory
spec.containers[].resources.requests.memory
```

### Ephemeral Storage

Container files, logs, and EmptyDir storage, as well as k8s cluster data, reside on the root filessystem of the host node. As storage is a limited resource, we may need to manage it as other resources. The scheduler will only choose a node with enough space to meet the sum of all the container requests. Shoud a particular container, or the sum of the containers in a Pod, use more than the limit, the Pod will be evicted.

```
spec.containers[].resources.limits.ephemeral-storage
spec.containers[].resources.requests.ephemeral-storage
```

## Using Label Selectors

