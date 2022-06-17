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


