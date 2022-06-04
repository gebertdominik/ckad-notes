# Kubernetes architecture

Introduction to kubernetes architecture - it's not directly mapped to the exam curriculum, but it's required to understand some basics about the architecture.

## Objectives

* Learn  common terms related to k8s
* Discuss history of k8s
* Understand what are control plane node components and worker node components
* Learn about Container Network Interface (CNI) configuration and Network Plugin

## What is Kubernetes?

According to k8s website it's an "open-source system for automating deployment, scaling, and management of contenerized applications". His precursor is Borg project, developed for 15 years by Google.

Kubernetes is written in Go Langiage - some claim it incorporates the best, whicle some claim the worst parts of C++, Python and Java

## Challenges

We need a CI pipeline to build, test, and verify our container images. Then we need an infrastucture on which we can run our containers and watch over them when things fail and self-heal. We also must be able to perform rollbacks, rolling updates and tear down resources when no longer needed.

All of those require flexible easy-to-manage network and storage. As containers are lunched on any worker node, the network must join the resource to other existing containers, while still keeping the trafic secure from otherrs. We also need a mainainable storage structure.

## Architecture
//TODO
![kubernetes_architecture.svg](https://myoctocat.com/assets/images/base-octocat.svg)


