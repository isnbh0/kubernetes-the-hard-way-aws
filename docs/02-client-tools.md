# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).


## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson`:

### OS X

```shell
brew install cfssl
```

### Linux

```shell
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.6.5/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.6.5/linux/cfssljson
```

```shell
chmod +x cfssl cfssljson
```

```shell
sudo mv cfssl cfssljson /usr/local/bin/
```

### Verification

Verify `cfssl` and `cfssljson` version 1.6.5 or higher is installed:

```shell
cfssl version
```

> output

```
Version: 1.6.5
Runtime: go1.22.0
```

```shell
cfssljson --version
```
```
Version: 1.6.5
Runtime: go1.22.0
```

## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

### OS X

```shell
brew install kubectl
```

### Linux

```shell
wget https://storage.googleapis.com/kubernetes-release/release/v1.31.0/bin/linux/amd64/kubectl
```

```
chmod +x kubectl
```

```
sudo mv kubectl /usr/local/bin/
```

### Verification

Verify `kubectl` version 1.31.0 or higher is installed:

```
kubectl version --client
```

> output

```
Client Version: v1.31.0
Kustomize Version: v5.4.2
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
