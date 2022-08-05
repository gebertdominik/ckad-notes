# Services & Networking (20%)

## Objectives

* Demonstrate basic understanding of Network Policies
* Provide and troubleshoot access to applications via services
* Use Ingress rules to expose applications

# Exposing Applications

* Use services to expose an application
* Understand the Cluster IP service
* Configure the NodePort service
* Deploy the LoadBalancer service
* Discuss the ExternalName service
* Understand an ingress controller


## Service Types

* `ClusterIP` - the `ClusterIP` service type is the default, and only provides access internally(except if manually creating an external endpoint). The range of `ClusterIP` used is defined via an API server startup option. The `kubectl proxy` command creates a local service to access a ClusterIP. This can be useful for troubleshooting or development work.

* `NodePort` - the `NodePort` type is great for debugging or when a static IP address is necessary, such as opening a particular address through a firewall. The NodePort range is defined in the cluster configuration.

* `LoadBalancer` - the `LoadBalancer` service was created to pass requests to a cloud provider like GKE or AWS. Private cloud solutions also may implement this service type if there is a cloud provider plugin, such as with CloudStack and OpenStack. Even without a cloud provider, the address is made available to public traffic, and packets are spread among the Pods in the deployment automatically.

* `Externalname` - a newer service is `ExternalName`, which is a bit different. It has no selectors, nor does it define ports or endpoints. It allows the retur of an alias to an external service. The redirection happens at the DNS level, not via a proxy or forward. This object can be useful for services not yet brought into the K8s cluster. A simple change of the type in the future would redirect traffic to the internal objects. As CoreDNS has become more stable, this service is not used as much.

## Services Diagram

The `kube-proxy` running on cluster nodes watches the API server service resources. It presents a type of virtual IP address for services other than `ExternalName`. The mode for this process has changed over versions of K8s.

In v1.0, services ran in `userspace` mode as TCP/UDP over IP or Layer 4. In the v1.1 release, the `iptables` proxy was added and became the default mode starting with v1.4


