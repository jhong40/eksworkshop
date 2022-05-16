
## Intermediate

<details>
  <summary>Pod Priority and Preemption</summary>
  
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
