# Beginner

+ https://github.com/jhong40/eksworkshop/blob/main/Beginner.md
+ https://github.com/jhong40/eksworkshop/blob/main/Intermediate.md
<details>
  <summary>IAM Groups</summary>
  
 ```
 POLICY=$(echo -n '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":"arn:aws-us-gov:iam::'; echo -n "$ACCOUNT_ID"; echo -n ':root"},"Action":"sts:AssumeRole","Condition":{}}]}')

echo ACCOUNT_ID=$ACCOUNT_ID
echo POLICY=$POLICY

aws iam create-role \
  --role-name k8sAdmin \
  --description "Kubernetes administrator role (for AWS IAM Authenticator for Kubernetes)." \
  --assume-role-policy-document "$POLICY" \
  --output text \
  --query 'Role.Arn'

aws iam create-role \
  --role-name k8sDev \
  --description "Kubernetes developer role (for AWS IAM Authenticator for Kubernetes)." \
  --assume-role-policy-document "$POLICY" \
  --output text \
  --query 'Role.Arn'
  
aws iam create-role \
  --role-name k8sInteg \
  --description "Kubernetes role for integration namespace in quick cluster." \
  --assume-role-policy-document "$POLICY" \
  --output text \
  --query 'Role.Arn'
  ```
 
  ```
  aws iam create-group --group-name k8sAdmin

ADMIN_GROUP_POLICY=$(echo -n '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAssumeOrganizationAccountRole",
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn'; echo -n "$AWS"; echo -n 'iam::'; echo -n "$ACCOUNT_ID"; echo -n ':role/k8sAdmin"
    }
  ]
}')
echo ADMIN_GROUP_POLICY=$ADMIN_GROUP_POLICY

aws iam put-group-policy \
--group-name k8sAdmin \
--policy-name k8sAdmin-policy \
--policy-document "$ADMIN_GROUP_POLICY"
```
  
```
aws iam create-group --group-name k8sDev
DEV_GROUP_POLICY=$(echo -n '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAssumeOrganizationAccountRole",
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
       "Resource": "arn'; echo -n "$AWS"; echo -n 'iam::'; echo -n "$ACCOUNT_ID"; echo -n ':role/k8sDev"
    }
  ]
}')
echo DEV_GROUP_POLICY=$DEV_GROUP_POLICY

aws iam put-group-policy \
--group-name k8sDev \
--policy-name k8sDev-policy \
--policy-document "$DEV_GROUP_POLICY"  
```  

```
aws iam create-group --group-name k8sInteg
INTEG_GROUP_POLICY=$(echo -n '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAssumeOrganizationAccountRole",
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
       "Resource": "arn'; echo -n "$AWS"; echo -n 'iam::'; echo -n "$ACCOUNT_ID"; echo -n ':role/k8sInteg"
    }
  ]
}')
echo INTEG_GROUP_POLICY=$INTEG_GROUP_POLICY

aws iam put-group-policy \
--group-name k8sInteg \
--policy-name k8sInteg-policy \
--policy-document "$INTEG_GROUP_POLICY"
```
  
```
aws iam list-groups  
```
```
aws iam create-user --user-name PaulAdmin
aws iam create-user --user-name JeanDev
aws iam create-user --user-name PierreInteg
aws iam add-user-to-group --group-name k8sAdmin --user-name PaulAdmin
aws iam add-user-to-group --group-name k8sDev --user-name JeanDev
aws iam add-user-to-group --group-name k8sInteg --user-name PierreInteg
aws iam get-group --group-name k8sAdmin
aws iam get-group --group-name k8sDev
aws iam get-group --group-name k8sInteg
aws iam create-access-key --user-name PaulAdmin | tee /tmp/PaulAdmin.json
aws iam create-access-key --user-name JeanDev | tee /tmp/JeanDev.json
aws iam create-access-key --user-name PierreInteg | tee /tmp/PierreInteg.json
```  

```
kubectl create namespace integration
kubectl create namespace development
```  
  
```  
cat << EOF | kubectl apply -f - -n development
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-role
rules:
  - apiGroups:
      - ""
      - "apps"
      - "batch"
      - "extensions"
    resources:
      - "configmaps"
      - "cronjobs"
      - "deployments"
      - "events"
      - "ingresses"
      - "jobs"
      - "pods"
      - "pods/attach"
      - "pods/exec"
      - "pods/log"
      - "pods/portforward"
      - "secrets"
      - "services"
    verbs:
      - "create"
      - "delete"
      - "describe"
      - "get"
      - "list"
      - "patch"
      - "update"
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-role-binding
subjects:
- kind: User
  name: dev-user
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
EOF
```
```
cat << EOF | kubectl apply -f - -n integration
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: integ-role
rules:
  - apiGroups:
      - ""
      - "apps"
      - "batch"
      - "extensions"
    resources:
      - "configmaps"
      - "cronjobs"
      - "deployments"
      - "events"
      - "ingresses"
      - "jobs"
      - "pods"
      - "pods/attach"
      - "pods/exec"
      - "pods/log"
      - "pods/portforward"
      - "secrets"
      - "services"
    verbs:
      - "create"
      - "delete"
      - "describe"
      - "get"
      - "list"
      - "patch"
      - "update"
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: integ-role-binding
subjects:
- kind: User
  name: integ-user
roleRef:
  kind: Role
  name: integ-role
  apiGroup: rbac.authorization.k8s.io
EOF
```  

```
eksctl create iamidentitymapping \
  --cluster eksworkshop-eksctl \
  --arn arn:aws:iam::${ACCOUNT_ID}:role/k8sDev \
  --username dev-user

eksctl create iamidentitymapping \
  --cluster eksworkshop-eksctl \
  --arn arn:aws:iam::${ACCOUNT_ID}:role/k8sInteg \
  --username integ-user

eksctl create iamidentitymapping \
  --cluster eksworkshop-eksctl \
  --arn arn:aws:iam::${ACCOUNT_ID}:role/k8sAdmin \
  --username admin \
  --group system:masters
kubectl get cm -n kube-system aws-auth -o yaml
eksctl get iamidentitymapping --cluster eksworkshop-eksctl 
```
```
# it can be used to delete entry
eksctl delete iamidentitymapping --cluster eksworkshop-eksctlv --arn arn:aws:iam::xxxxxxxxxx:role/k8sDev --username dev-user
```  

### TEST
```
mkdir -p ~/.aws

cat << EoF >> ~/.aws/config
[profile admin]
role_arn=arn:aws:iam::${ACCOUNT_ID}:role/k8sAdmin
source_profile=eksAdmin

[profile dev]
role_arn=arn:aws:iam::${ACCOUNT_ID}:role/k8sDev
source_profile=eksDev

[profile integ]
role_arn=arn:aws:iam::${ACCOUNT_ID}:role/k8sInteg
source_profile=eksInteg

EoF
```
```
cat << EoF >> ~/.aws/credentials

[eksAdmin]
aws_access_key_id=$(jq -r .AccessKey.AccessKeyId /tmp/PaulAdmin.json)
aws_secret_access_key=$(jq -r .AccessKey.SecretAccessKey /tmp/PaulAdmin.json)

[eksDev]
aws_access_key_id=$(jq -r .AccessKey.AccessKeyId /tmp/JeanDev.json)
aws_secret_access_key=$(jq -r .AccessKey.SecretAccessKey /tmp/JeanDev.json)

[eksInteg]
aws_access_key_id=$(jq -r .AccessKey.AccessKeyId /tmp/PierreInteg.json)
aws_secret_access_key=$(jq -r .AccessKey.SecretAccessKey /tmp/PierreInteg.json)

EoF
```  
  
```
### doesn't work
aws sts get-caller-identity --profile dev
  
```  
  
### Cleanup
```  
unset KUBECONFIG

kubectl delete namespace development integration
kubectl delete pod nginx-admin

eksctl delete iamidentitymapping --cluster eksworkshop-eksctl --arn arn:aws-us-gov:iam::${ACCOUNT_ID}:role/k8sAdmin
eksctl delete iamidentitymapping --cluster eksworkshop-eksctl --arn arn:aws-us-gov:iam::${ACCOUNT_ID}:role/k8sDev
eksctl delete iamidentitymapping --cluster eksworkshop-eksctl --arn arn:aws-us-gov:iam::${ACCOUNT_ID}:role/k8sInteg

aws iam remove-user-from-group --group-name k8sAdmin --user-name PaulAdmin
aws iam remove-user-from-group --group-name k8sDev --user-name JeanDev
aws iam remove-user-from-group --group-name k8sInteg --user-name PierreInteg

aws iam delete-group-policy --group-name k8sAdmin --policy-name k8sAdmin-policy 
aws iam delete-group-policy --group-name k8sDev --policy-name k8sDev-policy 
aws iam delete-group-policy --group-name k8sInteg --policy-name k8sInteg-policy 

aws iam delete-group --group-name k8sAdmin
aws iam delete-group --group-name k8sDev
aws iam delete-group --group-name k8sInteg

aws iam delete-access-key --user-name PaulAdmin --access-key-id=$(jq -r .AccessKey.AccessKeyId /tmp/PaulAdmin.json)
aws iam delete-access-key --user-name JeanDev --access-key-id=$(jq -r .AccessKey.AccessKeyId /tmp/JeanDev.json)
aws iam delete-access-key --user-name PierreInteg --access-key-id=$(jq -r .AccessKey.AccessKeyId /tmp/PierreInteg.json)

aws iam delete-user --user-name PaulAdmin
aws iam delete-user --user-name JeanDev
aws iam delete-user --user-name PierreInteg

aws iam delete-role --role-name k8sAdmin
aws iam delete-role --role-name k8sDev
aws iam delete-role --role-name k8sInteg

rm /tmp/*.json
rm /tmp/kubeconfig*

# reset aws credentials and config files
rm  ~/.aws/{config,credentials}
aws configure set default.region ${AWS_REGION}  
```    
</details>

<details>
  <summary>IAM roles for Service Account - IRSA</summary>
  
```
kubectl version --short
aws --version
aws eks describe-cluster --name eksworkshop-eksctl --query cluster.identity.oidc.issuer --output text
eksctl version
eksctl utils associate-iam-oidc-provider --cluster eksworkshop-eksctl --approve
```
```
aws iam list-policies --query 'Policies[?PolicyName==`AmazonS3ReadOnlyAccess`].Arn'
eksctl create iamserviceaccount \
    --name iam-test \
    --namespace default \
    --cluster eksworkshop-eksctl \
    --attach-policy-arn arn:$AWS:iam::aws:policy/AmazonS3ReadOnlyAccess \
    --approve \
    --override-existing-serviceaccounts
  
```  
```
kubectl get sa iam-test -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::XXXXX:role/eksctl-eksworkshop-eksctl-addon-iamserviceac-Role1-1M93WJN38Q39M
  creationTimestamp: "2022-05-11T16:27:43Z"
  labels:
    app.kubernetes.io/managed-by: eksctl
  name: iam-test
  namespace: default
  resourceVersion: "25270"
  uid: 54773bf8-4d5b-476f-b667-f586c7b8ad3e
secrets:
- name: iam-test-token-4cr5r  
```  
### TEST
  
```
mkdir ~/environment/irsa

cat <<EoF> ~/environment/irsa/job-s3.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: eks-iam-test-s3
spec:
  template:
    metadata:
      labels:
        app: eks-iam-test-s3
    spec:
      serviceAccountName: iam-test
      containers:
      - name: eks-iam-test
        image: amazon/aws-cli:latest
        args: ["s3", "ls"]
      restartPolicy: Never
EoF

kubectl apply -f ~/environment/irsa/job-s3.yaml
kubectl get job -l app=eks-iam-test-s3
kubectl logs -l app=eks-iam-test-s3
```  
```
cat <<EoF> ~/environment/irsa/job-ec2.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: eks-iam-test-ec2
spec:
  template:
    metadata:
      labels:
        app: eks-iam-test-ec2
    spec:
      serviceAccountName: iam-test
      containers:
      - name: eks-iam-test
        image: amazon/aws-cli:latest
        args: ["ec2", "describe-instances", "--region", "${AWS_REGION}"]
      restartPolicy: Never
  backoffLimit: 0
EoF

kubectl apply -f ~/environment/irsa/job-ec2.yaml
kubectl get job -l app=eks-iam-test-ec2
kubectl logs -l app=eks-iam-test-ec2  
```  
### Clean UP
```
kubectl delete -f ~/environment/irsa/job-s3.yaml
kubectl delete -f ~/environment/irsa/job-ec2.yaml

eksctl delete iamserviceaccount \
    --name iam-test \
    --namespace default \
    --cluster eksworkshop-eksctl \
    --wait

rm -rf ~/environment/irsa/
aws s3 rb s3://eksworkshop-$ACCOUNT_ID-$AWS_REGION --region $AWS_REGION --force
```  
</details>


<details>
  <summary>Security Group for Pods - CNI Plugin</summary>
  
```
mkdir ${HOME}/environment/sg-per-pod

cat << EoF > ${HOME}/environment/sg-per-pod/nodegroup-sec-group.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: eksworkshop-eksctl
  region: ${AWS_REGION}

managedNodeGroups:
- name: nodegroup-sec-group
  desiredCapacity: 1
  instanceType: m5.large
EoF

eksctl create nodegroup -f ${HOME}/environment/sg-per-pod/nodegroup-sec-group.yaml

kubectl get nodes --selector node.kubernetes.io/instance-type=m5.large
```
```
export VPC_ID=$(aws eks describe-cluster \
    --name eksworkshop-eksctl \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

# create RDS security group
aws ec2 create-security-group \
    --description 'RDS SG' \
    --group-name 'RDS_SG' \
    --vpc-id ${VPC_ID}

# save the security group ID for future use
export RDS_SG=$(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=RDS_SG Name=vpc-id,Values=${VPC_ID} \
    --query "SecurityGroups[0].GroupId" --output text)

echo "RDS security group ID: ${RDS_SG}"
```  
```
# create the POD security group
aws ec2 create-security-group \
    --description 'POD SG' \
    --group-name 'POD_SG' \
    --vpc-id ${VPC_ID}

# save the security group ID for future use
export POD_SG=$(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=POD_SG Name=vpc-id,Values=${VPC_ID} \
    --query "SecurityGroups[0].GroupId" --output text)

echo "POD security group ID: ${POD_SG}"
```
```
export NODE_GROUP_SG=$(aws ec2 describe-security-groups \
    --filters Name=tag:Name,Values=eks-cluster-sg-eksworkshop-eksctl-* Name=vpc-id,Values=${VPC_ID} \
    --query "SecurityGroups[0].GroupId" \
    --output text)
echo "Node Group security group ID: ${NODE_GROUP_SG}"

# allow POD_SG to connect to NODE_GROUP_SG using TCP 53
aws ec2 authorize-security-group-ingress \
    --group-id ${NODE_GROUP_SG} \
    --protocol tcp \
    --port 53 \
    --source-group ${POD_SG}

# allow POD_SG to connect to NODE_GROUP_SG using UDP 53
aws ec2 authorize-security-group-ingress \
    --group-id ${NODE_GROUP_SG} \
    --protocol udp \
    --port 53 \
    --source-group ${POD_SG}
```
```
# Cloud9 IP
export C9_IP=$(curl -s https://ipinfo.io/ip)

# allow Cloud9 to connect to RDS
aws ec2 authorize-security-group-ingress \
    --group-id ${RDS_SG} \
    --protocol tcp \
    --port 5432 \
    --cidr ${C9_IP}/32

# Allow POD_SG to connect to the RDS
aws ec2 authorize-security-group-ingress \
    --group-id ${RDS_SG} \
    --protocol tcp \
    --port 5432 \
    --source-group ${POD_SG}
```  
  
### Create RDS  
```
export PUBLIC_SUBNETS_ID=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=$VPC_ID" "Name=tag:Name,Values=eksctl-eksworkshop-eksctl-cluster/SubnetPublic*" \
    --query 'Subnets[*].SubnetId' \
    --output json | jq -c .)

# create a db subnet group
aws rds create-db-subnet-group \
    --db-subnet-group-name rds-eksworkshop \
    --db-subnet-group-description rds-eksworkshop \
    --subnet-ids ${PUBLIC_SUBNETS_ID}
```  
```
# get RDS SG ID
export RDS_SG=$(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=RDS_SG Name=vpc-id,Values=${VPC_ID} \
    --query "SecurityGroups[0].GroupId" --output text)

# generate a password for RDS
export RDS_PASSWORD="$(date | md5sum  |cut -f1 -d' ')"
echo ${RDS_PASSWORD}  > ~/environment/sg-per-pod/rds_password


# create RDS Postgresql instance
aws rds create-db-instance \
    --db-instance-identifier rds-eksworkshop \
    --db-name eksworkshop \
    --db-instance-class db.t3.micro \
    --engine postgres \
    --db-subnet-group-name rds-eksworkshop \
    --vpc-security-group-ids $RDS_SG \
    --master-username eksworkshop \
    --publicly-accessible \
    --master-user-password ${RDS_PASSWORD} \
    --backup-retention-period 0 \
    --allocated-storage 20
```  
##### check DB status: look for available
```
aws rds describe-db-instances \
    --db-instance-identifier rds-eksworkshop \
    --query "DBInstances[].DBInstanceStatus" \
    --output text
```
```
# get RDS endpoint
export RDS_ENDPOINT=$(aws rds describe-db-instances \
    --db-instance-identifier rds-eksworkshop \
    --query 'DBInstances[0].Endpoint.Address' \
    --output text)

echo "RDS endpoint: ${RDS_ENDPOINT}"
```
```
sudo amazon-linux-extras install -y postgresql12

cd sg-per-pod

cat << EoF > ~/environment/sg-per-pod/pgsql.sql
CREATE TABLE welcome (column1 TEXT);
insert into welcome values ('--------------------------');
insert into welcome values ('Welcome to the eksworkshop');
insert into welcome values ('--------------------------');
EoF

export RDS_PASSWORD=$(cat ~/environment/sg-per-pod/rds_password)

psql postgresql://eksworkshop:${RDS_PASSWORD}@${RDS_ENDPOINT}:5432/eksworkshop \
    -f ~/environment/sg-per-pod/pgsql.sql
```  
#### CNI configuration
```
############### ROLE_NAME is the role name for the new node
# Attach a new IAM policy the Node group role to allow the EC2 instances to manage network interfaces, their private IP addresses, and their attachment and detachment to and from instances.  
aws iam attach-role-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSVPCResourceController \
    --role-name ${ROLE_NAME}
```  
```
################ enable the CNI plugin to manage network interfaces for pods by setting the ENABLE_POD_ENI variable to true in the aws-node DaemonSet.  
kubectl -n kube-system set env daemonset aws-node ENABLE_POD_ENI=true

# let's way for the rolling update of the daemonset
kubectl -n kube-system rollout status ds aws-node
  
```  
```
kubectl get nodes --selector  eks.amazonaws.com/nodegroup=nodegroup-sec-group --show-labels
# ip-192-168-39-124.awsxxx.compute.internal   Ready    <none>   35m   v1.21.5-eks-9017834  ..... vpc.amazonaws.com/has-trunk-attached=true  
```  
```
#  A new Custom Resource Definition (CRD) has also been added automatically at the cluster creation  
kubectl get crd securitygrouppolicies.vpcresources.k8s.aws
```  
```
cat << EoF > ~/environment/sg-per-pod/sg-policy.yaml
apiVersion: vpcresources.k8s.aws/v1beta1
kind: SecurityGroupPolicy
metadata:
  name: allow-rds-access
spec:
  podSelector:
    matchLabels:
      app: green-pod
  securityGroups:
    groupIds:
      - ${POD_SG}
EoF
```
```
kubectl create namespace sg-per-pod
kubectl -n sg-per-pod apply -f ~/environment/sg-per-pod/sg-policy.yaml
kubectl -n sg-per-pod describe securitygrouppolicy
```  
#### Check the securitgrouppolicy
```
kubectl -n sg-per-pod get securitygrouppolicy -o yaml
  
apiVersion: v1
items:
- apiVersion: vpcresources.k8s.aws/v1beta1
  kind: SecurityGroupPolicy
  metadata:
    name: allow-rds-access
    namespace: sg-per-pod
  spec:
    podSelector:
      matchLabels:
        app: green-pod
    securityGroups:
      groupIds:
      - sg-0f82fc84a86128b20
```
#### Create secret to Connecto to DB
```
export RDS_PASSWORD=$(cat ~/environment/sg-per-pod/rds_password)

export RDS_ENDPOINT=$(aws rds describe-db-instances \
    --db-instance-identifier rds-eksworkshop \
    --query 'DBInstances[0].Endpoint.Address' \
    --output text)

kubectl create secret generic rds\
    --namespace=sg-per-pod \
    --from-literal="password=${RDS_PASSWORD}" \
    --from-literal="host=${RDS_ENDPOINT}"

kubectl -n sg-per-pod describe  secret rds
```
#### TEST  
```
cd ~/environment/sg-per-pod

curl -s -O https://www.eksworkshop.com/beginner/115_sg-per-pod/deployments.files/green-pod.yaml
curl -s -O https://www.eksworkshop.com/beginner/115_sg-per-pod/deployments.files/red-pod.yaml
```
#### TEST for green pod for SUCESS  
```
kubectl -n sg-per-pod apply -f ~/environment/sg-per-pod/green-pod.yaml
kubectl -n sg-per-pod rollout status deployment green-pod

export GREEN_POD_NAME=$(kubectl -n sg-per-pod get pods -l app=green-pod -o jsonpath='{.items[].metadata.name}')
kubectl -n sg-per-pod  logs -f ${GREEN_POD_NAME}
```
```
kubectl -n sg-per-pod  describe pod $GREEN_POD_NAME | head -11  
```
```
kubectl -n sg-per-pod  describe pod $GREEN_POD_NAME | head -11
# An ENI is attached to the pod.
# And the ENI has the security group POD_SG attached to it.  
  
Name:         green-pod-5b69b6f587-cfm9k
Namespace:    sg-per-pod
Priority:     0
Node:         ip-192-168-39-124.us-gov-west-1.compute.internal/192.168.39.124
Start Time:   Wed, 11 May 2022 19:46:37 +0000
Labels:       app=green-pod
              pod-template-hash=5b69b6f587
Annotations:  kubernetes.io/psp: eks.privileged
              vpc.amazonaws.com/pod-eni:
                [{"eniId":"eni-07c60b3834d5ddf15","ifAddress":"02:92:4d:8f:df:18","privateIp":"192.168.33.70","vlanId":1,"subnetCidr":"192.168.32.0/19"}]
Status:       Running  
```  
#### TEST for RED pod for FAILURE
```  
kubectl -n sg-per-pod apply -f ~/environment/sg-per-pod/red-pod.yaml
kubectl -n sg-per-pod rollout status deployment red-pod
export RED_POD_NAME=$(kubectl -n sg-per-pod get pods -l app=red-pod -o jsonpath='{.items[].metadata.name}')
kubectl -n sg-per-pod  logs -f ${RED_POD_NAME}
# Database connection failed due to timeout expired  
```
```
kubectl -n sg-per-pod  describe pod ${RED_POD_NAME} | head -11
```  
#### Clean UP
```
export VPC_ID=$(aws eks describe-cluster \
    --name eksworkshop-eksctl \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)
export RDS_SG=$(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=RDS_SG Name=vpc-id,Values=${VPC_ID} \
    --query "SecurityGroups[0].GroupId" --output text)
export POD_SG=$(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=POD_SG Name=vpc-id,Values=${VPC_ID} \
    --query "SecurityGroups[0].GroupId" --output text)
export C9_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
export NODE_GROUP_SG=$(aws ec2 describe-security-groups \
    --filters Name=tag:Name,Values=eks-cluster-sg-eksworkshop-eksctl-* Name=vpc-id,Values=${VPC_ID} \
    --query "SecurityGroups[0].GroupId" \
    --output text)

# uninstall the RPM package
# sudo yum remove -y $(sudo yum list installed | grep amzn2extra-postgresql12 | awk '{ print $1}')

# delete database
aws rds delete-db-instance \
    --db-instance-identifier rds-eksworkshop \
    --delete-automated-backups \
    --skip-final-snapshot

# delete kubernetes element
kubectl -n sg-per-pod delete -f ~/environment/sg-per-pod/green-pod.yaml
kubectl -n sg-per-pod delete -f ~/environment/sg-per-pod/red-pod.yaml
kubectl -n sg-per-pod delete -f ~/environment/sg-per-pod/sg-policy.yaml
kubectl -n sg-per-pod delete secret rds

# delete the namespace
kubectl delete ns sg-per-pod

# disable ENI trunking
kubectl -n kube-system set env daemonset aws-node ENABLE_POD_ENI=false
kubectl -n kube-system rollout status ds aws-node

# detach the IAM policy
aws iam detach-role-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSVPCResourceController \
    --role-name ${ROLE_NAME}

# remove the security groups rules
aws ec2 revoke-security-group-ingress \
    --group-id ${RDS_SG} \
    --protocol tcp \
    --port 5432 \
    --source-group ${POD_SG}

aws ec2 revoke-security-group-ingress \
    --group-id ${RDS_SG} \
    --protocol tcp \
    --port 5432 \
    --cidr ${C9_IP}/32

aws ec2 revoke-security-group-ingress \
    --group-id ${NODE_GROUP_SG} \
    --protocol tcp \
    --port 53 \
    --source-group ${POD_SG}

aws ec2 revoke-security-group-ingress \
    --group-id ${NODE_GROUP_SG} \
    --protocol udp \
    --port 53 \
    --source-group ${POD_SG}

# delete POD security group
aws ec2 delete-security-group \
    --group-id ${POD_SG}
```  
```
aws rds describe-db-instances \
    --db-instance-identifier rds-eksworkshop \
    --query "DBInstances[].DBInstanceStatus" \
    --output text
```
```
## Once the db deleted
# delete RDS SG
aws ec2 delete-security-group \
    --group-id ${RDS_SG}

# delete DB subnet group
aws rds delete-db-subnet-group \
    --db-subnet-group-name rds-eksworkshop
```
```
# delete the nodegroup
eksctl delete nodegroup -f ${HOME}/environment/sg-per-pod/nodegroup-sec-group.yaml --approve

# remove the trunk label
kubectl label node  --all 'vpc.amazonaws.com/has-trunk-attached'-

cd ~/environment
rm -rf sg-per-pod
```    
</details>
  
<details>
  <summary>Network Policy</summary>
  
#### Apply Calico Demonset
```
kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.6/config/v1.6/calico.yaml
kubectl get daemonset calico-node --namespace=kube-system  
```  
#### Create Resource
+ A namespace called **stars**
+ **frontend** and **backend** deployments and services within **stars** namespace
+ A namespace called **management-ui**
+ Deployment and service **management-ui** for the user interface seen on the browser, in the **management-ui** namespace
+ A namespace called **client**
+ **client** deployment and service in **client** namespace  
```
mkdir ~/environment/calico_resources
cd ~/environment/calico_resources
cd ~/environment/calico_resources
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/create_resources.files/namespace.yaml
kubectl apply -f namespace.yaml   #create stars 
```  
```
cd ~/environment/calico_resources
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/create_resources.files/management-ui.yaml
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/create_resources.files/backend.yaml
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/create_resources.files/frontend.yaml
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/create_resources.files/client.yaml
kubectl apply -f management-ui.yaml
kubectl apply -f backend.yaml
kubectl apply -f frontend.yaml
kubectl apply -f client.yaml
kubectl get pods --all-namespaces
```  
#### DEFAULT POD-TO-POD COMMUNICATION
```
kubectl get svc -o wide -n management-ui
# loadbalancer URL show all pods can communicate with each other  
```  
#### APPLY NETWORK POLICIES
```
cd ~/environment/calico_resources
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/apply_network_policies.files/default-deny.yaml
```  
```
cat default-deny.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny
spec:
  podSelector:
    matchLabels: {}
```
```
kubectl apply -n stars -f default-deny.yaml
kubectl apply -n client -f default-deny.yaml
## after appply, mgmt UI show nothing  
```  
```
cd ~/environment/calico_resources
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/apply_network_policies.files/allow-ui.yaml
wget https://eksworkshop.com/beginner/120_network-policies/calico/stars_policy_demo/apply_network_policies.files/allow-ui-client.yaml
```
```
cat allow-ui.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: stars
  name: allow-ui
spec:
  podSelector:
    matchLabels: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              role: management-ui

cat allow-ui-client.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: client
  name: allow-ui
spec:
  podSelector:
    matchLabels: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              role: management-ui
```
```
kubectl apply -f allow-ui.yaml
kubectl apply -f allow-ui-client.yaml
# After, you can see the 3 nodes in UI  (no traffic btw them)  
```
#### ALLOW DIRECTIONAL TRAFFIC
```
cd ~/environment/calico_resources
```
```
cat <<EOF | kubectl apply -f -
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: stars
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
EOF
```
```
cat <<EOF | kubectl apply -f -
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: stars
  name: frontend-policy
spec:
  podSelector:
    matchLabels:
      role: frontend
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              role: client
      ports:
        - protocol: TCP
          port: 80
EOF
```  
UI will show traffic   
#### Clean up
```
kubectl delete namespace client stars management-ui
```  
  
</details>  
  
<details>
  <summary>Expose Service: Classic LB, Network LB, APP LB</summary>
  
```
cat <<EoF > ~/environment/run-my-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  namespace: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EoF  
```
```
# create the namespace
kubectl create ns my-nginx
# create the nginx deployment with 2 replicas
kubectl -n my-nginx apply -f ~/environment/run-my-nginx.yaml
kubectl -n my-nginx get pods -o wide
kubectl -n my-nginx get pods -o yaml | grep 'podIP:'
``` 
```
kubectl -n my-nginx expose deployment/my-nginx
kubectl -n my-nginx get svc my-nginx
```  
```
# Create a variable set with the my-nginx service IP
export MyClusterIP=$(kubectl -n my-nginx get svc my-nginx -ojsonpath='{.spec.clusterIP}')
```
```  
# Create a new deployment and allocate a TTY for the container in the pod
kubectl -n my-nginx run -i --tty load-generator --env="MyClusterIP=${MyClusterIP}" --image=busybox /bin/sh
wget -q -O - ${MyClusterIP} | grep '<title>'   
exit
```  
#### Kubernetes supports 2 primary modes of finding a Service: Environment variables  and DNS
### Environment Var
```
kubectl -n my-nginx get pods -l run=my-nginx -o wide
export mypod=$(kubectl -n my-nginx get pods -l run=my-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl -n my-nginx exec ${mypod} -- printenv | grep SERVICE

KUBERNETES_SERVICE_HOST=10.100.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
# Note there’s no mention of your Service. This is because you created the replicas before the Service.  
```  
```
kubectl -n my-nginx rollout restart deployment my-nginx
kubectl -n my-nginx get pods -l run=my-nginx -o wide
export mypod=$(kubectl -n my-nginx get pods -l run=my-nginx -o jsonpath='{.items[0].metadata.name}')

kubectl -n my-nginx exec ${mypod} -- printenv | grep SERVICE
  
MY_NGINX_SERVICE_HOST=10.100.33.130
MY_NGINX_SERVICE_PORT=80
KUBERNETES_SERVICE_HOST=10.100.0.1
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443  
```  
### DNS
```  
kubectl get service -n kube-system -l k8s-app=kube-dns
kubectl -n my-nginx run curl --image=radial/busyboxplus:curl -i --tty
If you don't see a command prompt, try pressing enter.
[ root@curl:/ ]$ nslookup my-nginx
Server:    10.100.0.10
Address 1: 10.100.0.10 kube-dns.kube-system.svc.cluster.local

Name:      my-nginx
Address 1: 10.100.33.130 my-nginx.my-nginx.svc.cluster.local
```  
### Expose Service
```
kubectl -n my-nginx get svc my-nginx
kubectl -n my-nginx patch svc my-nginx -p '{"spec": {"type": "LoadBalancer"}}'
kubectl -n my-nginx get svc my-nginx
export loadbalancer=$(kubectl -n my-nginx get svc my-nginx -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')
curl -k -s http://${loadbalancer} | grep title 

kubectl -n my-nginx describe service my-nginx | grep Ingress
```
### AWS Load Balancer Controller
#### Prerequisite  
```
echo 'export LBC_VERSION="v2.4.1"' >>  ~/.bash_profile
echo 'export LBC_CHART_VERSION="1.4.1"' >>  ~/.bash_profile
.  ~/.bash_profile
  
if [ ! -x ${LBC_VERSION} ]
  then
    tput setaf 2; echo '${LBC_VERSION} has been set.'
  else
    tput setaf 1;echo '${LBC_VERSION} has NOT been set.'
fi
helm version --short
eksctl utils associate-iam-oidc-provider \
    --region ${AWS_REGION} \
    --cluster eksworkshop-eksctl \
    --approve
```  
#### IRSA
```
# AWSLoadBalancerControllerIAMPolicy  
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/${LBC_VERSION}/docs/install/iam_policy.json
sed -i "s/:aws:/:$AWS:/" iam_policy.json    ####################  
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
# Create a IAM role and ServiceAccoun
eksctl create iamserviceaccount \
  --cluster eksworkshop-eksctl \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve
```
  
#### Install the TargetGroupBinding CRDs
```
kubectl apply -k github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master
kubectl get crd
```
  
#### Deploy Helm chart
```
helm repo add eks https://aws.github.io/eks-charts

helm upgrade -i aws-load-balancer-controller \
    eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=eksworkshop-eksctl \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set image.tag="${LBC_VERSION}" \
    --version="${LBC_CHART_VERSION}"  \
    --set image.repository=013241004608.dkr.ecr.us-gov-west-1.amazonaws.com/amazon/aws-load-balancer-controller

# add image.repository to pull image depending on the region
  
kubectl -n kube-system rollout status deployment aws-load-balancer-controller
```  

#### Deploy Sample app
```
export EKS_CLUSTER_VERSION=$(aws eks describe-cluster --name eksworkshop-eksctl --query cluster.version --output text)

if [ "`echo "${EKS_CLUSTER_VERSION} < 1.19" | bc`" -eq 1 ]; then     
    curl -s https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/examples/2048/2048_full.yaml \
    | sed 's=alb.ingress.kubernetes.io/target-type: ip=alb.ingress.kubernetes.io/target-type: instance=g' \
    | kubectl apply -f -
fi

if [ "`echo "${EKS_CLUSTER_VERSION} >= 1.19" | bc`" -eq 1 ]; then     
    curl -s https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/examples/2048/2048_full_latest.yaml \
    | sed 's=alb.ingress.kubernetes.io/target-type: ip=alb.ingress.kubernetes.io/target-type: instance=g' \
    | kubectl apply -f -
fi  
```
```
kubectl get ingress/ingress-2048 -n game-2048    
export GAME_INGRESS_NAME=$(kubectl -n game-2048 get targetgroupbindings -o jsonpath='{.items[].metadata.name}')
kubectl -n game-2048 get targetgroupbindings ${GAME_INGRESS_NAME} -o yaml
```
```
export GAME_2048=$(kubectl get ingress/ingress-2048 -n game-2048 -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

echo http://${GAME_2048}
```
#### Clean UP
```
kubectl delete -f ~/environment/run-my-nginx.yaml
kubectl delete ns my-nginx
rm ~/environment/run-my-nginx.yaml

export EKS_CLUSTER_VERSION=$(aws eks describe-cluster --name eksworkshop-eksctl --query cluster.version --output text)

if [ "`echo "${EKS_CLUSTER_VERSION} < 1.19" | bc`" -eq 1 ]; then     
    curl -s https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/examples/2048/2048_full.yaml \
    | sed 's=alb.ingress.kubernetes.io/target-type: ip=alb.ingress.kubernetes.io/target-type: instance=g' \
    | kubectl delete -f -
fi

if [ "`echo "${EKS_CLUSTER_VERSION} >= 1.19" | bc`" -eq 1 ]; then     
    curl -s https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/examples/2048/2048_full_latest.yaml \
    | sed 's=alb.ingress.kubernetes.io/target-type: ip=alb.ingress.kubernetes.io/target-type: instance=g' \
    | kubectl delete -f -
fi

unset EKS_CLUSTER_VERSION

helm uninstall aws-load-balancer-controller \
    -n kube-system

kubectl delete -k github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master

eksctl delete iamserviceaccount \
    --cluster eksworkshop-eksctl \
    --name aws-load-balancer-controller \
    --namespace kube-system \
    --wait

aws iam delete-policy \
    --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy
```  
</details>  

<details>
  <summary>Stateful Set: EBS volume</summary>
  
  Container Storage Interface (CSI) - Amazon EBS CSI Driver
  ```
 export EBS_CSI_POLICY_NAME="Amazon_EBS_CSI_Driver"

mkdir ${HOME}/environment/ebs_statefulset
cd ${HOME}/environment/ebs_statefulset

# download the IAM policy document
curl -sSL -o ebs-csi-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-ebs-csi-driver/master/docs/example-iam-policy.json
sed -i "s/:aws:/:$AWS:/" ebs-csi-policy.json    #################### 
# Create the IAM policy
aws iam create-policy \
  --region ${AWS_REGION} \
  --policy-name ${EBS_CSI_POLICY_NAME} \
  --policy-document file://${HOME}/environment/ebs_statefulset/ebs-csi-policy.json

# export the policy ARN as a variable
export EBS_CSI_POLICY_ARN=$(aws --region ${AWS_REGION} iam list-policies --query 'Policies[?PolicyName==`'$EBS_CSI_POLICY_NAME'`].Arn' --output text)
```
```
# Create an IAM OIDC provider for your cluster
eksctl utils associate-iam-oidc-provider \
  --region=$AWS_REGION \
  --cluster=eksworkshop-eksctl \
  --approve

# Create a service account
eksctl create iamserviceaccount \
  --cluster eksworkshop-eksctl \
  --name ebs-csi-controller-irsa \
  --namespace kube-system \
  --attach-policy-arn $EBS_CSI_POLICY_ARN \
  --override-existing-serviceaccounts \
  --approve
```
#### Deploy the Amazon EBS CSI Driver  
```
# add the aws-ebs-csi-driver as a helm repo
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver

# search for the driver
helm search  repo aws-ebs-csi-driver
  
helm upgrade --install aws-ebs-csi-driver \
  --version=1.2.4 \
  --namespace kube-system \
  --set serviceAccount.controller.create=false \
  --set serviceAccount.snapshot.create=false \
  --set enableVolumeScheduling=true \
  --set enableVolumeResizing=true \
  --set enableVolumeSnapshot=true \
  --set serviceAccount.snapshot.name=ebs-csi-controller-irsa \
  --set serviceAccount.controller.name=ebs-csi-controller-irsa \
  aws-ebs-csi-driver/aws-ebs-csi-driver

kubectl -n kube-system rollout status deployment ebs-csi-controller 
```  
#### Define Storage Class
```
cat << EoF > ${HOME}/environment/ebs_statefulset/mysql-storageclass.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: mysql-gp2
provisioner: ebs.csi.aws.com # Amazon EBS CSI driver
parameters:
  type: gp2
  encrypted: 'true' # EBS volumes will always be encrypted by default
volumeBindingMode: WaitForFirstConsumer # EBS volumes are AZ specific
reclaimPolicy: Delete
mountOptions:
- debug
EoF
```
```
kubectl create -f ${HOME}/environment/ebs_statefulset/mysql-storageclass.yaml  
kubectl describe storageclass mysql-gp2
```  
#### Create ConfigMap
```
kubectl create namespace mysql
cd ${HOME}/environment/ebs_statefulset

cat << EoF > ${HOME}/environment/ebs_statefulset/mysql-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  namespace: mysql
  labels:
    app: mysql
data:
  master.cnf: |
    # Apply this config only on the leader.
    [mysqld]
    log-bin
  slave.cnf: |
    # Apply this config only on followers.
    [mysqld]
    super-read-only
EoF

kubectl create -f ${HOME}/environment/ebs_statefulset/mysql-configmap.yaml
```  
#### Create Services
```
cat << EoF > ${HOME}/environment/ebs_statefulset/mysql-services.yaml
# Headless service for stable DNS entries of StatefulSet members.
apiVersion: v1
kind: Service
metadata:
  namespace: mysql
  name: mysql
  labels:
    app: mysql
spec:
  ports:
  - name: mysql
    port: 3306
  clusterIP: None
  selector:
    app: mysql
---
# Client service for connecting to any MySQL instance for reads.
# For writes, you must instead connect to the leader: mysql-0.mysql.
apiVersion: v1
kind: Service
metadata:
  namespace: mysql
  name: mysql-read
  labels:
    app: mysql
spec:
  ports:
  - name: mysql
    port: 3306
  selector:
    app: mysql
EoF

kubectl create -f ${HOME}/environment/ebs_statefulset/mysql-services.yaml
```
#### Create StatefulSet
```
cd ${HOME}/environment/ebs_statefulset
wget https://eksworkshop.com/beginner/170_statefulset/statefulset.files/mysql-statefulset.yaml
kubectl apply -f ${HOME}/environment/ebs_statefulset/mysql-statefulset.yaml
kubectl -n mysql rollout status statefulset mysql
# kubectl -n mysql get pods -l app=mysql --watch
kubectl -n mysql get pvc -l app=mysql
```  
#### Test MySql
```
kubectl -n mysql run mysql-client --image=mysql:5.7 -i --rm --restart=Never --\
  mysql -h mysql-0.mysql <<EOF
CREATE DATABASE test;
CREATE TABLE test.messages (message VARCHAR(250));
INSERT INTO test.messages VALUES ('hello, from mysql-client');
EOF
```
```
kubectl -n mysql run mysql-client --image=mysql:5.7 -it --rm --restart=Never --\
  mysql -h mysql-read -e "SELECT * FROM test.messages"
```
```
kubectl -n mysql run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --\
   bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
```  
#### Test Failure
```
kubectl -n mysql exec mysql-1 -c mysql -- mv /usr/bin/mysql /usr/bin/mysql.off
kubectl -n mysql get pod mysql-1
# NAME      READY   STATUS    RESTARTS   AGE
# mysql-1   1/2     Running   0          8m42s  
  
```  
```
kubectl -n mysql run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --\
   bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
```  
###### Revert back
```
kubectl -n mysql exec mysql-1 -c mysql -- mv /usr/bin/mysql.off /usr/bin/mysql
kubectl -n mysql get pod mysql-1
#NAME      READY   STATUS    RESTARTS   AGE
#mysql-1   2/2     Running   0          11m  
```
###### Failed Pod
```
kubectl -n mysql delete pod mysql-1
kubectl -n mysql get pod mysql-1 -w
```
###### Test Sacaling
```
kubectl -n mysql scale statefulset mysql --replicas=3
kubectl -n mysql rollout status statefulset mysql
# another terminal check 
kubectl -n mysql run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --\
   bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
  
kubectl -n mysql run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
 mysql -h mysql-2.mysql -e "SELECT * FROM test.messages"

kubectl -n mysql  scale statefulset mysql --replicas=2
kubectl -n mysql get pods -l app=mysql

kubectl -n mysql  get pvc -l app=mysql  # still see 3 pv  (doesn't delete pv)
kubectl -n mysql delete pvc data-mysql-2
  
```  
#### Clean UP
```
export EBS_CSI_POLICY_NAME="Amazon_EBS_CSI_Driver"
export EBS_CSI_POLICY_ARN=$(aws --region ${AWS_REGION} iam list-policies --query 'Policies[?PolicyName==`'${EBS_CSI_POLICY_NAME}'`].Arn' --output text)

kubectl delete \
  -f ${HOME}/environment/ebs_statefulset/mysql-statefulset.yaml \
  -f ${HOME}/environment/ebs_statefulset/mysql-services.yaml \
  -f ${HOME}/environment/ebs_statefulset/mysql-configmap.yaml \
  -f ${HOME}/environment/ebs_statefulset/mysql-storageclass.yaml

# Delete the mysql namespace 
kubectl delete namespace mysql

# Uninstall the aws-ebs-csi-driver
helm -n kube-system uninstall aws-ebs-csi-driver

# Delete the service account
eksctl delete iamserviceaccount \
  --cluster eksworkshop-eksctl \
  --namespace kube-system \
  --name ebs-csi-controller-irsa \
  --wait

# Delete the IAM Amazon_EBS_CSI_Driver policy
aws iam delete-policy \
  --region ${AWS_REGION} \
  --policy-arn ${EBS_CSI_POLICY_ARN}

cd ${HOME}/environment
rm -rf ${HOME}/environment/ebs_statefulset
```    
</details>  
 
