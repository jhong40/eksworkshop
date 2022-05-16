
## Intermediate

<details>
  <summary>Resource Management</summary>
  
  
### Resource limit 
+ Enforce minimum and maximum compute resources usage per Pod or Container in a namespace.
+ Enforce minimum and maximum storage request per PersistentVolumeClaim in a namespace.
+ Enforce a ratio between request and limit for a resource in a namespace.
+ Set default request/limit for compute resources in a namespace and automatically inject them to Containers at runtime.
```
cat <<EoF > ~/environment/resource-management/low-usage-limit-range.yml
apiVersion: v1
kind: LimitRange
metadata:
  name: low-usage-range
spec:
  limits:
  - max:
      cpu: 1
      memory: 300M 
    min:
      cpu: 0.5
      memory: 100M
    type: Container
EoF

kubectl apply -f ~/environment/resource-management/low-usage-limit-range.yml --namespace low-usage


cat <<EoF > ~/environment/resource-management/high-usage-limit-range.yml
apiVersion: v1
kind: LimitRange
metadata:
  name: high-usage-range
spec:
  limits:
  - max:
      cpu: 2
      memory: 2G 
    min:
      cpu: 1
      memory: 1G
    type: Container
EoF

kubectl apply -f ~/environment/resource-management/high-usage-limit-range.yml --namespace high-usage
```  
```
## Set default
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
  
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
  
```  
### Resource Quota
+ One ResourceQuota for each namespace.
+ Users create resources (pods, services, etc.) in the namespace, and the quota system tracks usage to ensure it does not exceed hard resource limits defined in a ResourceQuota.
+ If creating or updating a resource violates a quota constraint, the request will fail with HTTP status code 403 FORBIDDEN with a message explaining the constraint that would have been violated.
+ If quota is enabled in a namespace for compute resources like cpu and memory, users must specify requests or limits for those values; otherwise, the quota system may reject pod creation. Hint: Use the LimitRanger admission controller to force defaults for pods that make no compute resource requirements.  
```
# Create different namespaces
kubectl create namespace blue
kubectl create namespace red

kubectl create quota blue-team --hard=limits.cpu=1,limits.memory=1G --namespace blue
kubectl create quota red-team --hard=services.loadbalancers=1 --namespace red  
```
  
### Pod Priority and Preemption
 ```
 cat <<EoF > ~/environment/resource-management/high-priority-class.yml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100
globalDefault: false
description: "High-priority Pods"
EoF

kubectl apply -f ~/environment/resource-management/high-priority-class.yml


cat <<EoF > ~/environment/resource-management/low-priority-class.yml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 50
globalDefault: false
description: "Low-priority Pods"
EoF

kubectl apply -f ~/environment/resource-management/low-priority-class.yml

 ```
 ```
 cat <<EoF > ~/environment/resource-management/low-priority-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 50
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      priorityClassName: "low-priority"      
      containers:            
       - image: nginx
         name: nginx-deployment
         resources:
           limits:
              memory: 1G  
EoF
kubectl apply -f ~/environment/resource-management/low-priority-deployment.yml
kubectl get deployment nginx-deployment --watch 
 ``` 
 ```
 cat <<EoF > ~/environment/resource-management/high-priority-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: high-nginx-deployment
  name: high-nginx-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: high-nginx-deployment
  template:
    metadata:
      labels:
        app: high-nginx-deployment
    spec:
      priorityClassName: "high-priority"      
      containers:            
       - image: nginx
         name: high-nginx-deployment
         resources:
           limits:
              memory: 1G
EoF
kubectl apply -f ~/environment/resource-management/high-priority-deployment.yml
 ``` 
  
```
kubectl get deployment  --watch

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   21/50   50           21          2m15s

high-nginx-deployment   0/5     0            0           0s
high-nginx-deployment   0/5     0            0           0s
high-nginx-deployment   0/5     0            0           0s
high-nginx-deployment   0/5     5            0           0s
nginx-deployment        20/50   49           20          4m9s
nginx-deployment        20/50   50           20          4m9s
nginx-deployment        19/50   49           19          4m9s
nginx-deployment        19/50   50           19          4m9s
nginx-deployment        18/50   49           18          4m9s
nginx-deployment        18/50   50           18          4m9s
nginx-deployment        17/50   49           17          4m9s
nginx-deployment        17/50   50           17          4m9s
nginx-deployment        16/50   49           16          4m9s
nginx-deployment        16/50   50           16          4m9s
high-nginx-deployment   1/5     5            1           8s
high-nginx-deployment   2/5     5            2           8s
high-nginx-deployment   3/5     5            3           22s
high-nginx-deployment   4/5     5            4           23s
high-nginx-deployment   5/5     5            5           23s
```  
  
 </details>
