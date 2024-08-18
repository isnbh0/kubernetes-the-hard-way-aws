# Deploying the DNS Cluster Add-on

In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery, backed by [CoreDNS](https://coredns.io/), to applications running inside the Kubernetes cluster.

## The DNS Cluster Add-on

Deploy the `coredns` cluster add-on:

```sh
cat <<EOF > coredns-values.yaml
service:
  clusterIP: 10.32.0.10  # Use the IP you've designated for DNS in your cluster
replicaCount: 2
isClusterService: true
serviceType: "ClusterIP"
plugins:
  kubernetes:
    enabled: true
  prometheus:
    enabled: true
  errors:
    enabled: true
  log:
    enabled: true
EOF

helm repo add coredns https://coredns.github.io/helm
helm repo update
helm install coredns coredns/coredns -f coredns-values.yaml -n kube-system
```

> output

```sh
NAME: coredns
LAST DEPLOYED: Mon Aug 19 14:14:56 2024
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CoreDNS is now running in the cluster as a cluster-service.

It can be tested with the following:

1. Launch a Pod with DNS tools:

kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools

2. Query the DNS server:

/ # host kubernetes
```

List the pods created by the `core-dns` deployment:

```sh
kubectl get pods -l k8s-app=coredns -n kube-system
```

> output

```sh
NAME                       READY   STATUS    RESTARTS   AGE
coredns-8494f9c688-hh7r2   1/1     Running   0          10s
coredns-8494f9c688-zqrj2   1/1     Running   0          10s
```

## Verification

Create a `busybox` deployment:

```sh
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```

List the pod created by the `busybox` deployment:

```sh
kubectl get pods -l run=busybox
```

> output

```sh
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          3s
```

Retrieve the full name of the `busybox` pod:

```sh
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
echo $POD_NAME
```

Execute a DNS lookup for the `kubernetes` service inside the `busybox` pod:

```sh
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

> output

```sh
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```

Next: [Smoke Test](13-smoke-test.md)
