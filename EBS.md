# CNI configuration
### ROLE_NAME is the role name for the new node
# Attach a new IAM policy the Node group role to allow the EC2 instances to manage network interfaces, their private IP addresses, and their attachment and detachment to and from instances.  
```
aws iam attach-role-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSVPCResourceController \
    --role-name ${ROLE_NAME}
```
### enable the CNI plugin to manage network interfaces for pods by setting the ENABLE_POD_ENI variable to true in the aws-node DaemonSet.  
```
kubectl -n kube-system set env daemonset aws-node ENABLE_POD_ENI=true

# let's way for the rolling update of the daemonset
kubectl -n kube-system rollout status ds aws-node
```
