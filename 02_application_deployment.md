# Application Environment, Configuration and Security

## Objectives

* Discover and use resources that extend Kubernetes (CRD)
* Understand authentication, authorization and admission control
* Understanding and defining resource requirements, limits and quotas
* Understand ConfigMaps
* Create & consume Secrets
* Understand ServiceAccounts
* Understand SecurityContext


# Deployment Configuration

## Objectives

* Understand and create persistent volumes
* Configure persistent volume claims
* Manage volume access modes
* Deploy an application with access to persistent storage
* Discuss the dynamic provisioning of storage
* Configure secrets and ConfigMaps
* Update a deployed application
* Roll back to previous version

## Volumes Overview

Containers engines have traditionally not offered stoarage that outlives container. A K8s volume shares at least the Pod lifetime, not the containers within. Should a container terminate, the data would continue to be available to the new container. A volume can persist longer than a Pod, and can be accessed by multiple Pods using **PersistentVolumeClaims**. This allows for state persistency.

A volume is a directory made avaliable to containers in a Pod. As of 1.14, there were 28 different volume types ranging from `rbd` for Ceph, to `NFS`, to dynamic volumes from a cloud provider like Google's `gcePersistentDisk`. Each has particular configuration options and dependencies.

Should we want our storage lifetime to be distinct from a Pod, we can use Persistent Volumes. These allow for empty or pre-populated volumes to be claimed by a Pod using a Persistent Volume Claim, then outlive the Pod.

There are two API Objects which exists to provide data to a Pod already. Encoded data can be passed using a Secret and non-encoded data can be passed with a **ConfigMap**. These can be used to pass important data like SSH keys, passwords, or even a config gile like `etc/hosts`.

We should be aware that any capacity mentioned does not represent an actual limit on the amount of space a container can consume. Should a volume have a capacity of 10G, that does not mean there is 10G of backend storage available. If there is more and an application were to continue to write it, there is no block on how much space is used, with the possibleexception of a newer limit on ephemeral storage usage. A new CSI driver has become available, so at least we can track actual usage.

## Volume Spec

One of the many types of storage available is an `emptyDir`. The kubelet will create the directory in the container, but not mount any storage. Any data created is written to the shared container space. As a result, it would not be persistent storage. When the Pod is destroyed, the directory would be deleted along with the container.

```
apiVersion: v1
kind: Pod
metadata:
    name: busybox
    namespace: default
spec:
    containers:
    - image: busybox
      name: busy
      command:
        - sleep
        - "3600"
      volumeMounts:
      - mountPath: /scratch
        name: scratch-volume
    volumes:
    - name: scratch-volume
            emptyDir: {}

```

The YAML file above would create a Pod with a single container with a volume named `scratch-volume` created, which would create the `/scratch` directory inside the container.

## Volume Types

### GCEpersistentDisk and awsElasticBlockStore

In GCE or AWS we can use volumes of type `GCEpersistentDisk` or `awsElasticBlockStore` which allows us to mount GCE and EBS disks in our Pods, assuming we have already set up accounts and privileges.

### emptyDir and hostPath

`emptyDir` and `hostPath` volumes are easy to use. As mentioned, `emptyDir` is an empty directory that gets erased when the Pod dies, but is recreated when the container restarts. `hostPath` volume mounts a resource from the host node filesystem. The resource could be a directory, file socket, character, or block device. These resources must already exist on the host to be used. There are two types, `DirectoryOrCreate` and `FileOrCreate` which create the resources on the host, and use them if they don't already exist.

### NFS and iSCSI

`NFS`(Network File System) and `iSCSI`(Internet Small Computer System Interface) are straightforward choices for multiple readers scenarios.

### rbd, CephFS and GlusterFS

`rbd` for block storage or `CephFS` and `GlusterFS` if available in our k8s cluster, can be a good choice for multiple writer needs

### Other Volume types

* `azureDisk`
* `azureFile`
* `csi`
* `downwardAPI`
* `fc` (fibre channel)
* `flocker`
* `gitRepo`
* `local`
* `projected`
* `portworxVolume`
* `quobyte`
* `scaleIO`
* `secret`
* `storageos`
* `vsphereVolume`
* `persistentVolumeClaim`
* ...

## Shared Volume Example

The following YAML file, for Pod ExampleA creates a Pod with two containers, one called alphacont, the other called betacont both with access to a shared volume called sharedvol:

```
  containers:
   - name: alphacont
     image: busybox
     volumeMounts:
     - mountPath: /alphadir
       name: sharevol
   - name: betacont
     image: busybox
     volumeMounts:
     - mountPath: /betadir
       name: sharevol
   volumes:
   - name: sharevol
     emptyDir: {}   
```

```
$ kubectl exec -ti exampleA -c betacont -- touch /betadir/foobar
$ kubectl exec -ti exampleA -c alphacont -- ls -l /alphadir

total 0
-rw-r--r-- 1 root root 0 Nov 19 16:26 foobar

```

We can use `emptyDir` or `hostPath` easily - they don't require any additional setup. Note that one container wrote, and the other container had immediate access to the data. There is nothing to keep the containers from overwriting the other's data. Locking or versioning considerations must be part of the application to avoid corruption.

## Persistent Volumes and Claims

A `PersistentVolume`(PV) is a storage abstraction used to retain data longer than the Pod using it. Pods define a volume of type `PersistentVolumeCLaim`(PVC) with various parameters for size and possibly the type of backend storage known as its `StorageClass`. The cluster then attaches the `PersistentVolume`.

K8s will dynamically use volumes that are available, irrespective of its storage type, allowing claims to any backend storage.

```
kubectl get pv
kubectl get pvc
```

## Phases to Persistent Storage

```mermaid
flowchart LR
   Provisioning --> Binding --> Using --> Releasing --> Reclaiming
```
