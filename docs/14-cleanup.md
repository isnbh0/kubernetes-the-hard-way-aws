# Cleaning Up

In this lab you will delete the compute resources created during this tutorial.

## Compute Instances

Delete the all worker instances, then afterwards delete controller instances:

```sh
echo "Issuing shutdown to worker nodes.. " && \
aws ec2 terminate-instances \
  --instance-ids \
    $(aws ec2 describe-instances --filters \
      "Name=tag:Name,Values=worker-0,worker-1,worker-2" \
      "Name=instance-state-name,Values=running" \
      --output text --query 'Reservations[].Instances[].InstanceId')

echo "Waiting for worker nodes to finish terminating.. " && \
aws ec2 wait instance-terminated \
  --instance-ids \
    $(aws ec2 describe-instances \
      --filter "Name=tag:Name,Values=worker-0,worker-1,worker-2" \
      --output text --query 'Reservations[].Instances[].InstanceId')

echo "Issuing shutdown to master nodes.. " && \
aws ec2 terminate-instances \
  --instance-ids \
    $(aws ec2 describe-instances --filter \
      "Name=tag:Name,Values=controller-0,controller-1,controller-2" \
      "Name=instance-state-name,Values=running" \
      --output text --query 'Reservations[].Instances[].InstanceId')

echo "Waiting for master nodes to finish terminating.. " && \
aws ec2 wait instance-terminated \
  --instance-ids \
    $(aws ec2 describe-instances \
      --filter "Name=tag:Name,Values=controller-0,controller-1,controller-2" \
      --output text --query 'Reservations[].Instances[].InstanceId')

aws ec2 delete-key-pair --key-name kubernetes
```

## Networking

Delete the external load balancer network resources:

```sh
ensure_var LOAD_BALANCER_ARN
ensure_var TARGET_GROUP_ARN
ensure_var SECURITY_GROUP_ID
ensure_var ROUTE_TABLE_ID
ensure_var NLB_ROUTE_TABLE_ID
ensure_var INTERNET_GATEWAY_ID
ensure_var SUBNET_ID
ensure_var NLB_SUBNET_ID
ensure_var VPC_ID

aws elbv2 delete-load-balancer --load-balancer-arn "${LOAD_BALANCER_ARN}"
aws elbv2 delete-target-group --target-group-arn "${TARGET_GROUP_ARN}"
aws ec2 delete-security-group --group-id "${SECURITY_GROUP_ID}"
ROUTE_TABLE_ASSOCIATION_ID="$(aws ec2 describe-route-tables \
  --route-table-ids "${ROUTE_TABLE_ID}" \
  --output text --query 'RouteTables[].Associations[].RouteTableAssociationId')"
aws ec2 disassociate-route-table --association-id "${ROUTE_TABLE_ASSOCIATION_ID}"
NLB_ROUTE_TABLE_ASSOCIATION_ID="$(aws ec2 describe-route-tables \
  --route-table-ids "${NLB_ROUTE_TABLE_ID}" \
  --output text --query 'RouteTables[].Associations[].RouteTableAssociationId')"
aws ec2 disassociate-route-table --association-id "${NLB_ROUTE_TABLE_ASSOCIATION_ID}"
aws ec2 delete-route-table --route-table-id "${NLB_ROUTE_TABLE_ID}"
aws ec2 delete-route-table --route-table-id "${ROUTE_TABLE_ID}"
echo "Waiting a minute for all public address(es) to be unmapped.. " && sleep 60

aws ec2 detach-internet-gateway \
  --internet-gateway-id "${INTERNET_GATEWAY_ID}" \
  --vpc-id "${VPC_ID}"
aws ec2 delete-internet-gateway --internet-gateway-id "${INTERNET_GATEWAY_ID}"
aws ec2 delete-subnet --subnet-id "${NLB_SUBNET_ID}"
aws ec2 delete-subnet --subnet-id "${SUBNET_ID}"
aws ec2 delete-vpc --vpc-id "${VPC_ID}"
```


## Session File

```sh
echo "Current contents of .kubernetes_session_vars:"
cat .kubernetes_session_vars
echo "Clearing contents of .kubernetes_session_vars.."
rm .kubernetes_session_vars
touch .kubernetes_session_vars
echo "Done."
```


## Cleanup kubeconfig

```sh
kubectl config delete-cluster kubernetes-the-hard-way
kubectl config delete-context kubernetes-the-hard-way
kubectl config delete-user admin
```
