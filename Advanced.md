## Advanced

<details>
  <summary>ISTIO</summary>
   
#### DOWNLOAD AND INSTALL ISTIO CLI  
```  
export ISTIO_VERSION="1.10.0"

cd ~/environment
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=${ISTIO_VERSION} sh -

cd ${HOME}/environment/istio-${ISTIO_VERSION}
sudo cp -v bin/istioctl /usr/local/bin/
istioctl version --remote=false
```  
#### INSTALL ISTIO
```
yes | istioctl install --set profile=demo
kubectl -n istio-system get svc
kubectl -n istio-system get pods
```
#### DEPLOY SAMPLE APPS
```
kubectl create namespace bookinfo
kubectl label namespace bookinfo istio-injection=enabled    # allow Istio to automatically inject the Sidecar Proxy.
kubectl get ns bookinfo --show-labels
  
kubectl -n bookinfo apply \
  -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/platform/kube/bookinfo.yaml
kubectl -n bookinfo get pod,svc

## Create an Istio Gateway
kubectl -n bookinfo \
 apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/bookinfo-gateway.yaml
## wait for a few min  
export GATEWAY_URL=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "http://${GATEWAY_URL}/productpage"
```  
## TRAFFIC MANAGEMENT
#### Create the default destination rules 
```
kubectl -n bookinfo apply \
  -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/destination-rule-all.yaml
kubectl -n bookinfo get destinationrules -o yaml
```
#### Route traffic to one version of a service reviews:v1
```
kubectl -n bookinfo \
  apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl -n bookinfo get virtualservices reviews -o yaml 
```  
```
### output  
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1  
```  
## Route based on user identity
```
kubectl -n bookinfo \
  apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
kubectl -n bookinfo get virtualservices reviews -o yaml
```
```
## output
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
## Click Sign in from the top right corner of the page.
Log in using jason as user name with a blank password.
You will only see reviews:v2 all the time. Others will see reviews:v1.  
```
## Injecting an HTTP delay fault
```
kubectl -n bookinfo \
  apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
kubectl -n bookinfo get virtualservice ratings -o yaml
```
```
# output
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        fixedDelay: 7s
        percentage:
          value: 100
    match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```
```
kubectl -n bookinfo \
  apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
kubectl -n bookinfo get virtualservice ratings -o yaml
```
```
spec:
  hosts:
  - ratings
  http:
  - fault:
      abort:
        httpStatus: 500
        percentage:
          value: 100
    match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
 ```
### Traffic Shifting
```
kubectl -n bookinfo \
  apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/virtual-service-all-v1.yaml
##  transfer 50% of the traffic from reviews:v1 to reviews:v3  
kubectl -n bookinfo \
  apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
kubectl -n bookinfo get virtualservice reviews -o yaml
```
```
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
```  
```
## decide review 3 is good. 100% traffic -> reviews:v3 (red colored star ratings
kubectl -n bookinfo apply -f ${HOME}/environment/istio-${ISTIO_VERSION}/samples/bookinfo/networking/virtual-service-reviews-v3.yaml
```  
## MONITOR & VISUALIZE  
#### Install Grafana and Prometheus
```
export ISTIO_RELEASE=$(echo $ISTIO_VERSION |cut -d. -f1,2)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-${ISTIO_RELEASE}/samples/addons/prometheus.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-${ISTIO_RELEASE}/samples/addons/grafana.yaml
kubectl -n istio-system get deploy grafana prometheus
```  
#### Install Jaeger and Kiali
```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-${ISTIO_RELEASE}/samples/addons/jaeger.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-${ISTIO_RELEASE}/samples/addons/kiali.yaml   ## has issue
kubectl -n istio-system get deploy jaeger kiali
```  
#### Generate traffic to collect telemetry data
```
export GATEWAY_URL=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
watch --interval 1 curl -s -I -XGET "http://${GATEWAY_URL}/productpage"
``` 
#### Launch Kiali
```
kubectl -n istio-system port-forward \
$(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 8080:20001
```
  
  
  
  
</details>
  
  
  
