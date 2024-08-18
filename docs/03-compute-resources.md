# Provisioning Compute Resources

[Guide](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md)

## Networking

### VPC

```sh
ensure_var VPC_ID
if [ $? -ne 0 ]; then
    VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --output text --query 'Vpc.VpcId')
    set_var VPC_ID "$VPC_ID"
    echo "VPC_ID=${VPC_ID}"
    aws ec2 create-tags --resources "${VPC_ID}" --tags Key=Name,Value=kubernetes-the-hard-way
    aws ec2 modify-vpc-attribute --vpc-id "${VPC_ID}" --enable-dns-support '{"Value": true}'
    aws ec2 modify-vpc-attribute --vpc-id "${VPC_ID}" --enable-dns-hostnames '{"Value": true}'
else
    echo "Using existing VPC: ${VPC_ID}"
fi
```

### Subnet

```sh
ensure_var VPC_ID
ensure_var SUBNET_ID
if [ $? -ne 0 ]; then
    SUBNET_ID=$(aws ec2 create-subnet \
      --vpc-id "${VPC_ID}" \
      --cidr-block 10.0.1.0/24 \
      --availability-zone ap-northeast-2a \
      --output text --query 'Subnet.SubnetId')
    set_var SUBNET_ID "$SUBNET_ID"
    echo "SUBNET_ID=${SUBNET_ID}"
    aws ec2 create-tags --resources "${SUBNET_ID}" --tags Key=Name,Value=kubernetes
else
    echo "Using existing Subnet: ${SUBNET_ID}"
fi
```

### Internet Gateway

```sh
ensure_var VPC_ID
ensure_var INTERNET_GATEWAY_ID
if [ $? -ne 0 ]; then
    INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --output text --query 'InternetGateway.InternetGatewayId')
    set_var INTERNET_GATEWAY_ID "$INTERNET_GATEWAY_ID"
    echo "INTERNET_GATEWAY_ID=${INTERNET_GATEWAY_ID}"
    aws ec2 create-tags --resources "${INTERNET_GATEWAY_ID}" --tags Key=Name,Value=kubernetes
    aws ec2 attach-internet-gateway --internet-gateway-id "${INTERNET_GATEWAY_ID}" --vpc-id "${VPC_ID}"
else
    echo "Using existing Internet Gateway: ${INTERNET_GATEWAY_ID}"
fi
```


### Route Tables

```sh
ensure_var VPC_ID
ensure_var SUBNET_ID
ensure_var INTERNET_GATEWAY_ID
ensure_var ROUTE_TABLE_ID
if [ $? -ne 0 ]; then
    ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id ${VPC_ID} --output text --query 'RouteTable.RouteTableId')
    set_var ROUTE_TABLE_ID "$ROUTE_TABLE_ID"
    echo "ROUTE_TABLE_ID=${ROUTE_TABLE_ID}"
    aws ec2 create-tags --resources "${ROUTE_TABLE_ID}" --tags Key=Name,Value=kubernetes
    aws ec2 associate-route-table --route-table-id "${ROUTE_TABLE_ID}" --subnet-id "${SUBNET_ID}"
    aws ec2 create-route --route-table-id "${ROUTE_TABLE_ID}" --destination-cidr-block 0.0.0.0/0 --gateway-id "${INTERNET_GATEWAY_ID}"
else
    echo "Using existing Route Table: ${ROUTE_TABLE_ID}"
fi
```

### Security Groups (aka Firewall Rules)

```sh
ensure_var VPC_ID
ensure_var SECURITY_GROUP_ID
if [ $? -ne 0 ]; then
    SECURITY_GROUP_ID=$(aws ec2 create-security-group \
      --group-name kubernetes \
      --description "Kubernetes security group" \
      --vpc-id ${VPC_ID} \
      --output text --query 'GroupId')
    set_var SECURITY_GROUP_ID "$SECURITY_GROUP_ID"
    echo "SECURITY_GROUP_ID=${SECURITY_GROUP_ID}"
    aws ec2 create-tags --resources ${SECURITY_GROUP_ID} --tags Key=Name,Value=kubernetes
    aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol all --cidr 10.0.0.0/16
    aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol all --cidr 10.200.0.0/16
    aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 22 --cidr 0.0.0.0/0
    aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 6443 --cidr 0.0.0.0/0
    aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 443 --cidr 0.0.0.0/0
    aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol icmp --port -1 --cidr 0.0.0.0/0
else
    echo "Using existing Security Group: ${SECURITY_GROUP_ID}"
fi
```

### Kubernetes Public Access - Create a Network Load Balancer

```sh
ensure_var NLB_SUBNET_ID
ensure_var NLB_ROUTE_TABLE_ID
if [ $? -ne 0 ]; then
    NLB_SUBNET_ID=$(aws ec2 create-subnet \
      --vpc-id "${VPC_ID}" \
      --cidr-block 10.0.2.0/24 \
      --availability-zone ap-northeast-2a \
      --output text --query 'Subnet.SubnetId')
    set_var NLB_SUBNET_ID "$NLB_SUBNET_ID"
    echo "NLB_SUBNET_ID=${NLB_SUBNET_ID}"
    # Create a route table for the NLB subnet
    NLB_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id ${VPC_ID} --output text --query 'RouteTable.RouteTableId')
    set_var NLB_ROUTE_TABLE_ID "$NLB_ROUTE_TABLE_ID"
    echo "NLB_ROUTE_TABLE_ID=${NLB_ROUTE_TABLE_ID}"
    aws ec2 create-tags --resources "${NLB_ROUTE_TABLE_ID}" --tags Key=Name,Value=kubernetes-nlb
    # Associate the NLB subnet with the route table
    aws ec2 associate-route-table --route-table-id "${NLB_ROUTE_TABLE_ID}" --subnet-id "${NLB_SUBNET_ID}"
    # Add a route to the internet gateway for the NLB subnet
    aws ec2 create-route --route-table-id "${NLB_ROUTE_TABLE_ID}" --destination-cidr-block 0.0.0.0/0 --gateway-id "${INTERNET_GATEWAY_ID}"
    echo "NLB Route Table created and associated with NLB Subnet"
else
    echo "Using existing NLB Subnet: ${NLB_SUBNET_ID}"
    echo "Using existing NLB Route Table: ${NLB_ROUTE_TABLE_ID}"
fi

ensure_var LOAD_BALANCER_ARN
ensure_var TARGET_GROUP_ARN
if [ $? -ne 0 ]; then
    LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer \
      --name kubernetes \
      --subnets ${NLB_SUBNET_ID} \
      --scheme internet-facing \
      --type network \
      --output text --query 'LoadBalancers[].LoadBalancerArn')
    set_var LOAD_BALANCER_ARN "$LOAD_BALANCER_ARN"
    echo "LOAD_BALANCER_ARN=${LOAD_BALANCER_ARN}"
    TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
      --name kubernetes \
      --protocol TCP \
      --port 6443 \
      --vpc-id ${VPC_ID} \
      --target-type ip \
      --output text --query 'TargetGroups[].TargetGroupArn')
    set_var TARGET_GROUP_ARN "$TARGET_GROUP_ARN"
    echo "TARGET_GROUP_ARN=${TARGET_GROUP_ARN}"
    # We'll register the targets after creating the instances
    aws elbv2 create-listener \
      --load-balancer-arn ${LOAD_BALANCER_ARN} \
      --protocol TCP \
      --port 443 \
      --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} \
      --output text --query 'Listeners[].ListenerArn'
else
    echo "Using existing Load Balancer: ${LOAD_BALANCER_ARN}"
    echo "Using existing Target Group: ${TARGET_GROUP_ARN}"
fi
```

```sh
ensure_var LOAD_BALANCER_ARN
ensure_var KUBERNETES_PUBLIC_ADDRESS
if [ $? -ne 0 ]; then
    KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
      --load-balancer-arns ${LOAD_BALANCER_ARN} \
      --output text --query 'LoadBalancers[].DNSName')
    set_var KUBERNETES_PUBLIC_ADDRESS "$KUBERNETES_PUBLIC_ADDRESS"
    echo "KUBERNETES_PUBLIC_ADDRESS=${KUBERNETES_PUBLIC_ADDRESS}"
else
    echo "Using existing Kubernetes Public Address: ${KUBERNETES_PUBLIC_ADDRESS}"
fi
```

## Compute Instances

### Instance Image

```sh
ensure_var IMAGE_ID
if [ $? -ne 0 ]; then
    IMAGE_ID=$(aws ec2 describe-images --owners 099720109477 \
      --output json \
      --filters \
      'Name=root-device-type,Values=ebs' \
      'Name=architecture,Values=arm64' \
      'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-arm64-server-*' \
      | jq -r '.Images|sort_by(.Name)[-1]|.ImageId')
    set_var IMAGE_ID "$IMAGE_ID"
    echo "IMAGE_ID=${IMAGE_ID}"
else
    echo "Using existing Image: ${IMAGE_ID}"
fi
```

### SSH Key Pair

```sh
aws ec2 create-key-pair --key-name kubernetes --output text --query 'KeyMaterial' > kubernetes.id_rsa
chmod 600 kubernetes.id_rsa
```

### Kubernetes Controllers

Using `t4g.micro` instances

```sh
for i in 0 1 2; do
  private_ip="10.0.1.1${i}"

  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name kubernetes \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t4g.micro \
    --private-ip-address ${private_ip} \
    --user-data "name=controller-${i}" \
    --subnet-id ${SUBNET_ID} \
    --block-device-mappings='{"DeviceName": "/dev/sda1", "Ebs": { "VolumeSize": 50 }, "NoDevice": "" }' \
    --output text --query 'Instances[].InstanceId')

  aws ec2 modify-instance-attribute --instance-id ${instance_id} --no-source-dest-check
  aws ec2 create-tags --resources ${instance_id} --tags "Key=Name,Value=controller-${i}"

  # Register the instance with the target group using private IP
  aws elbv2 register-targets --target-group-arn ${TARGET_GROUP_ARN} --targets Id=${private_ip}

  echo "controller-${i} created with IP ${private_ip} and registered with target group"
done
```

### Kubernetes Workers

```sh
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name kubernetes \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t4g.micro \
    --private-ip-address 10.0.1.2${i} \
    --user-data "name=worker-${i}|pod-cidr=10.200.${i}.0/24" \
    --subnet-id ${SUBNET_ID} \
    --block-device-mappings='{"DeviceName": "/dev/sda1", "Ebs": { "VolumeSize": 50 }, "NoDevice": "" }' \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute --instance-id ${instance_id} --no-source-dest-check
  aws ec2 create-tags --resources ${instance_id} --tags "Key=Name,Value=worker-${i}"
  echo "worker-${i} created"
done
```

Next: [Certificate Authority](04-certificate-authority.md)
