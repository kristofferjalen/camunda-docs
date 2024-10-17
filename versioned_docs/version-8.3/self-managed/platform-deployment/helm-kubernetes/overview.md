---
id: overview
title: "Camunda 8 on Kubernetes"
sidebar_label: "Overview"
description: "An overview of Kubernetes, its environments, and officially supported platforms"
---

We strongly recommend using Kubernetes and Helm to deploy and run Camunda 8 in production.

There are many ways you can provision and configure a Kubernetes cluster, and there are a number of architectural choices you need to make. Will your workers run in the Kubernetes cluster or external to it? You will need to configure your Kubernetes cluster and modify this to suit the architecture you are building.

## Kubernetes environments

You can install Camunda 8 on your Kubernetes environment of choice, e.g.:

- [Stock Kubernetes](https://kubernetes.io/docs/).
- [Kubernetes KIND](https://github.com/kubernetes-sigs/kind), Minikube, K3s, and MicroK8s for local development.
- [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift), an enterprise ready solution with a focus on security.
- Cloud service providers like Google GKE, Azure AKS, Amazon EKS, etc.

## Officially supported platforms

With the right configuration, Camunda 8 can be deployed on any Kubernetes distribution (Cloud or on-premises). However, we officially test and support a [specific list of platforms](./platforms/platforms.md).

## Versioning

Starting from July 2023 (v8.2.8), the Camunda 8 **Helm chart** version follows the same unified schema
and schedule as [Camunda 8 applications](https://github.com/camunda/camunda-platform) (e.g., if the application version is 8.2.8, then chart version is also 8.2.8).

For more details about the applications version included in the Helm chart, review the [full version matrix](https://helm.camunda.io/camunda-platform/version-matrix/).

## Use Helm to install on Kubernetes

There are several alternatives to deploy applications to a Kubernetes cluster, but we recommend using our provided Helm charts to deploy a set of components into your cluster. Helm allows you to choose exactly what chart (set of components) you want to install and how these components need to be configured.

At [helm.camunda.io](https://helm.camunda.io/), you'll find a Helm chart to configure a three-broker cluster with two Elasticsearch instances, Operate, two Zeebe Gateways and Tasklist. This size is comparable with the Production-S cluster plan in [Camunda 8 SaaS](https://camunda.com/get-started/). It should be sufficient for 80% of use cases.

Refer to the [documentation on Camunda's Helm charts](./deploy.md) for details.

To do, you must have the following tools installed in your local environment:

- `kubectl`: Kubernetes Control CLI tool, installed and connected to your cluster
- `helm`: Kubernetes Helm CLI tool

## Useful tools related to Camunda

- **Camunda Desktop Modeler**: to model/modify business processes. [Learn more](/components/modeler/desktop-modeler/index.md).
- **Zeebe CTL (`zbctl`)**: command line tool to interact with a Zeebe cluster (local/remote). You can get the `zbctl` tool from the official [Zeebe release page](https://github.com/camunda-cloud/zeebe/releases).