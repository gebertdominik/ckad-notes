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

The use of Docker had been the industry standard. Large companies like Red Hat are moving to other open source tools. While currently new, we can expect them to become the new standard in the future. Those new tools are for example:

* Buildah

It's a tool that focuses on creating Open Container Initiative (OCI) images. It allows for creating images with and without Dockerfile.

* Podman

A "Pod manager" tool, allows for the life cycle management of a container. We could consider it as a replacement for docker. Red Hat's goal was to replace all Docker functionallity with podman, using the same syntax to minimize transition issues. Some places alias docker to actually run podman.

## Revrite Legacy Applications

