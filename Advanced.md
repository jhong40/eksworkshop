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
  
  
  
