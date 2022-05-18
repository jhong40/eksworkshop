
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
## Set default (Not in EKSworkshop)
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

<details>
  <summary>Deploy Jenkins</summary>
  
```
aws codecommit create-repository --repository-name eksworkshop-app  
```  
```
aws iam create-user \
  --user-name git-user

aws iam attach-user-policy \
  --user-name git-user \
  --policy-arn arn:aws:iam::aws:policy/AWSCodeCommitPowerUser

aws iam create-service-specific-credential \
  --user-name git-user --service-name codecommit.amazonaws.com \
  | tee /tmp/gituser_output.json

GIT_USERNAME=$(cat /tmp/gituser_output.json | jq -r '.ServiceSpecificCredential.ServiceUserName')
GIT_PASSWORD=$(cat /tmp/gituser_output.json | jq -r '.ServiceSpecificCredential.ServicePassword')
CREDENTIAL_ID=$(cat /tmp/gituser_output.json | jq -r '.ServiceSpecificCredential.ServiceSpecificCredentialId') 
```  
```
sudo pip install git-remote-codecommit

git clone codecommit::${AWS_REGION}://eksworkshop-app
cd eksworkshop-app  
```  
```
cat << EOF > server.go

package main

import (
    "fmt"
    "net/http"
)

func helloWorld(w http.ResponseWriter, r *http.Request){
    fmt.Fprintf(w, "Hello World")
}

func main() {
    http.HandleFunc("/", helloWorld)
    http.ListenAndServe(":8080", nil)
}
EOF
```
```
cat << EOF > server_test.go

package main

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

func Test_helloWorld(t *testing.T) {
	req, err := http.NewRequest("GET", "http://domain.com/", nil)
	if err != nil {
		t.Fatal(err)
	}

	res := httptest.NewRecorder()
	helloWorld(res, req)

	exp := "Hello World"
	act := res.Body.String()
	if exp != act {
		t.Fatalf("Expected %s got %s", exp, act)
	}
}

EOF
```
```
cat << EOF > Jenkinsfile
pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: golang
    image: golang:1.13
    command:
    - cat
    tty: true
"""
    }
  }
  stages {
    stage('Run tests') {
      steps {
        container('golang') {
          sh 'go test'
        }
      }
    }
    stage('Build') {
        steps {
            container('golang') {
              sh 'go build -o eksworkshop-app'
              archiveArtifacts "eksworkshop-app"
            }
            
        }
    }
    
  }
}

EOF
```
```
git add --all && git commit -m "Initial commit." && git push
cd ~/environment
```  
```
eksctl utils associate-iam-oidc-provider --cluster eksworkshop-eksctl --approve
	
eksctl create iamserviceaccount \
    --name jenkins \
    --namespace default \
    --cluster eksworkshop-eksctl \
    --attach-policy-arn arn:$AWS:iam::aws:policy/AWSCodeCommitPowerUser \
    --approve \
    --override-existing-serviceaccounts
```	
### Deploy Jenkins
```
cat << EOF > values.yaml
---
controller:
  # Used for label app.kubernetes.io/component
  componentName: "jenkins-controller"
  image: "jenkins/jenkins"
  tag: "2.289.2-lts-jdk11"
  additionalPlugins:
    - aws-codecommit-jobs:0.3.0
    - aws-java-sdk:1.11.995
    - junit:1.51
    - ace-editor:1.1
    - workflow-support:3.8
    - pipeline-model-api:1.8.5
    - pipeline-model-definition:1.8.5
    - pipeline-model-extensions:1.8.5
    - workflow-job:2.41
    - credentials-binding:1.26
    - aws-credentials:1.29
    - credentials:2.5
    - lockable-resources:2.11
    - branch-api:2.6.4
  resources:
    requests:
      cpu: "1024m"
      memory: "4Gi"
    limits:
      cpu: "4096m"
      memory: "8Gi"
  javaOpts: "-Xms4000m -Xmx4000m"
  servicePort: 80
  serviceType: LoadBalancer
agent:
  Enabled: false
rbac:
  create: true
serviceAccount:
  create: false
  name: "jenkins"
EOF
```
```
helm install cicd stable/jenkins -f values.yaml
kubectl get pods -w
	
```	
</details>  


<details>
  <summary>Loggin with Amazon OpenSearch, Fluent Bit</summary>

```
eksctl utils associate-iam-oidc-provider \
    --cluster eksworkshop-eksctl \
    --approve
```
```
mkdir ~/environment/logging/

export ES_DOMAIN_NAME="eksworkshop-logging"

cat <<EoF > ~/environment/logging/fluent-bit-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "es:ESHttp*"
            ],
            "Resource": "arn:$AWS:es:${AWS_REGION}:${ACCOUNT_ID}:domain/${ES_DOMAIN_NAME}",
            "Effect": "Allow"
        }
    ]
}
EoF

aws iam create-policy   \
  --policy-name fluent-bit-policy \
  --policy-document file://~/environment/logging/fluent-bit-policy.json

kubectl create namespace logging

eksctl create iamserviceaccount \
    --name fluent-bit \
    --namespace logging \
    --cluster eksworkshop-eksctl \
    --attach-policy-arn "arn:$AWS:iam::${ACCOUNT_ID}:policy/fluent-bit-policy" \
    --approve \
    --override-existing-serviceaccounts

kubectl -n logging describe sa fluent-bit

```	
#### PROVISION AN AMAZON OPENSEARCH CLUSTER
Fine-grained access control offers two forms of authentication and authorization:

+ A built-in user database, which makes it easy to configure usernames and passwords inside of Amazon OpenSearch cluster.
+ AWS Identity and Access Management (IAM) integration, which lets you map IAM principals to permissions.	
```
# name of our Amazon OpenSearch cluster
export ES_DOMAIN_NAME="eksworkshop-logging"

# Elasticsearch version
export ES_VERSION="OpenSearch_1.0"

# OpenSearch Dashboards admin user
export ES_DOMAIN_USER="eksworkshop"

# OpenSearch Dashboards admin password
export ES_DOMAIN_PASSWORD="$(openssl rand -base64 12)_Ek1$"

# Download and update the template using the variables created previously
curl -sS https://www.eksworkshop.com/intermediate/230_logging/deploy.files/es_domain.json \
  | envsubst > ~/environment/logging/es_domain.json
sed -i 's/:aws:/:aws-us-gov:/' ~/environment/logging/es_domain.json    #######
	
# Create the cluster
aws opensearch create-domain \
  --cli-input-json  file://~/environment/logging/es_domain.json	
```  
#### Check OpenSearch creation status
```
if [ $(aws opensearch describe-domain --domain-name ${ES_DOMAIN_NAME} --query 'DomainStatus.Processing') == "false" ]
  then
    tput setaf 2; echo "The Amazon OpenSearch cluster is ready"
  else
    tput setaf 1;echo "The Amazon OpenSearch cluster is NOT ready"
fi	
```
#### Configure Amazon OpenSearch Acess -  Maping role
```
# We need to retrieve the Fluent Bit Role ARN
export FLUENTBIT_ROLE=$(eksctl get iamserviceaccount --cluster eksworkshop-eksctl --namespace logging -o json | jq '.[].status.roleARN' -r)

# Get the Amazon OpenSearch Endpoint
export ES_ENDPOINT=$(aws opensearch describe-domain --domain-name ${ES_DOMAIN_NAME} --output text --query "DomainStatus.Endpoint")

# Update the Elasticsearch internal database
curl -sS -u "${ES_DOMAIN_USER}:${ES_DOMAIN_PASSWORD}" \
    -X PATCH \
    https://${ES_ENDPOINT}/_opendistro/_security/api/rolesmapping/all_access?pretty \
    -H 'Content-Type: application/json' \
    -d'
[
  {
    "op": "add", "path": "/backend_roles", "value": ["'${FLUENTBIT_ROLE}'"]
  }
]
'
```	
#### Deploy Fluent Bit
```
cd ~/environment/logging

# get the Amazon OpenSearch Endpoint
export ES_ENDPOINT=$(aws es describe-elasticsearch-domain --domain-name ${ES_DOMAIN_NAME} --output text --query "DomainStatus.Endpoint")

curl -Ss https://www.eksworkshop.com/intermediate/230_logging/deploy.files/fluentbit.yaml \
    | envsubst > ~/environment/logging/fluentbit.yaml

kubectl apply -f ~/environment/logging/fluentbit.yaml
kubectl --namespace=logging get pods
```
#### OpenSearch Dashboard
```
echo "OpenSearch Dashboards URL: https://${ES_ENDPOINT}/_dashboards/
OpenSearch Dashboards user: ${ES_DOMAIN_USER}
OpenSearch Dashboards password: ${ES_DOMAIN_PASSWORD}"
```	
#### Clean UP
```
cd  ~/environment/

kubectl delete -f ~/environment/logging/fluentbit.yaml

aws opensearch delete-domain \
    --domain-name ${ES_DOMAIN_NAME}

eksctl delete iamserviceaccount \
    --name fluent-bit \
    --namespace logging \
    --cluster eksworkshop-eksctl \
    --wait

aws iam delete-policy   \
  --policy-arn "arn:$aws:iam::${ACCOUNT_ID}:policy/fluent-bit-policy"

kubectl delete namespace logging

rm -rf ~/environment/logging

unset ES_DOMAIN_NAME
unset ES_VERSION
unset ES_DOMAIN_USER
unset ES_DOMAIN_PASSWORD
unset FLUENTBIT_ROLE
unset ES_ENDPOINT
```	
</details>  
<details>
  <summary>MONITORING USING PROMETHEUS AND GRAFANA</summary>
  
```
# add prometheus Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# add grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts	
```	
```
kubectl create namespace prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```
#### Deploy Prometheus	
```
# ip add # get eth0 ip address
# kubectl port-forward -n prometheus  deploy/prometheus-server 9090:9090	
kubectl port-forward --address 10.0.2.15 -n prometheus  deploy/prometheus-server 9090:9090
http://localhost:9090/targets	
```	
#### Deploy Grafana
```
mkdir ${HOME}/environment/grafana

cat << EoF > ${HOME}/environment/grafana/grafana.yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
EoF

kubectl create namespace grafana

helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values ${HOME}/environment/grafana/grafana.yaml \
    --set service.type=LoadBalancer

kubectl get all -n grafana
```
#### Get the Grafana UI and pass
```
export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "http://$ELB"

kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```	
#### Clean UP
```
helm uninstall prometheus --namespace prometheus
kubectl delete ns prometheus

helm uninstall grafana --namespace grafana
kubectl delete ns grafana

rm -rf ${HOME}/environment/grafana
```	
</details>	

