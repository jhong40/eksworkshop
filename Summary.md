# Sesurity Group 


##### Attach a new IAM policy the Node group role to allow the EC2 instances to manage network interfaces, their private IP addresses, and their attachment and detachment to and from instances.  ROLE_NAME is the role name for the new node
```
aws iam attach-role-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSVPCResourceController \
    --role-name ${ROLE_NAME}
```
#### Enable the CNI plugin to manage network interfaces for pods by setting the ENABLE_POD_ENI variable to true in the aws-node DaemonSet.  
```
kubectl -n kube-system set env daemonset aws-node ENABLE_POD_ENI=true

# let's way for the rolling update of the daemonset
kubectl -n kube-system rollout status ds aws-node

#  A new Custom Resource Definition (CRD) has also been added automatically at the cluster creation  
kubectl get crd securitygrouppolicies.vpcresources.k8s.aws
```
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
