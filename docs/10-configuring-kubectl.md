# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```sh
ensure_var LOAD_BALANCER_ARN
KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
--load-balancer-arns ${LOAD_BALANCER_ARN} \
--output text --query 'LoadBalancers[].DNSName')

kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${KUBERNETES_PUBLIC_ADDRESS}:443

kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem

kubectl config set-context kubernetes-the-hard-way \
  --cluster=kubernetes-the-hard-way \
  --user=admin

kubectl config use-context kubernetes-the-hard-way
```

## Verification

Check the version of the remote Kubernetes cluster:

```sh
kubectl version
```

> output

```sh
Client Version: v1.31.0
Kustomize Version: v5.4.2
Server Version: v1.31.0
```

List the nodes in the remote Kubernetes cluster:

```sh
kubectl get nodes
```

> output

```sh
NAME           STATUS   ROLES    AGE   VERSION
ip-10-0-1-20   Ready    <none>   61s   v1.31.0
ip-10-0-1-21   Ready    <none>   62s   v1.31.0
ip-10-0-1-22   Ready    <none>   61s   v1.31.0
```

Next: [Provisioning Pod Network Routes](11-pod-network-routes.md)
