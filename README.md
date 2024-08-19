# About

This is a fork of the awesome [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by Kelsey Hightower and is geared towards using it on AWS. Also thanks to [@slawekzachcial](https://github.com/slawekzachcial) for his [work](https://github.com/slawekzachcial/kubernetes-the-hard-way-aws) that made this easier, and further thanks to [@prabhatsharma](https://github.com/prabhatsharma) for his [work](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws) that made this even easier. (Yes, this is a fork of a fork of a fork.)


# Kubernetes The Hard Way (AWS 2024-08)

This tutorial walks you through setting up Kubernetes the hard way. This guide is not for people looking for a fully automated command to bring up a Kubernetes cluster. If that's you then check out [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine), [AWS Elastic Container Service for Kubernetes](https://aws.amazon.com/eks/) or the [Getting Started Guides](https://kubernetes.io/docs/setup).

Kubernetes The Hard Way is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a Kubernetes cluster.

> The results of this tutorial should not be viewed as production ready, and may receive limited support from the community, but don't let that stop you from learning!

## Target Audience

The target audience for this tutorial is someone planning to support a production Kubernetes cluster and wants to understand how everything fits together.

## Cluster Details

Kubernetes The Hard Way guides you through bootstrapping a highly available Kubernetes cluster with end-to-end encryption between components and RBAC authentication.

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.31.0
* [containerd](https://github.com/containerd/containerd) v1.7.20
* [coredns](https://github.com/coredns/coredns) v1.32.0
* [cni](https://github.com/containernetworking/cni) v1.5.1
* [etcd](https://github.com/etcd-io/etcd) v3.5.15

## Labs

This tutorial is a fork of the official [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) guide, tailored for use on AWS.

Please note that this fork is of an older version of the original guide, and while it has been upgraded to work with the latest versions of the referenced tools(as of 2024-08), its content has diverged somewhat from the original guide.

Also, as I prepared this repository while I was myself studying the material, the quality of the added code in this fork may not be up to standard. Please expect some rough edges and hopefully some fixes in the future.

* [Prerequisites](docs/01-prerequisites.md)
* [Installing the Client Tools](docs/02-client-tools.md)
* [Provisioning Compute Resources](docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Smoke Test](docs/13-smoke-test.md)
* [Cleaning Up](docs/14-cleanup.md)
