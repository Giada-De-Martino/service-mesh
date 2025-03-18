# Service Mesh Demo with Istio and Go

## Prerequisites
- Minikube
- Istioctl
- Docker
- Kubernetes CLI (kubectl)

## Setup Instructions
### 1. Start Minikube
```sh
minikube start
```

### 2. Install Istio
```sh
istioctl install -f demo-profile-no-gateways.yaml -y
```
and add a namespace :
```sh
kubectl label namespace default istio-injection=enabled
```
### 3. Install the Kubernetes Gateway API CRDs
```sh
kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
{ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.2.1" | kubectl apply -f -; }
```

### 4. Deploy the demo application
```sh
kubectl apply -f platform/kube/bookinfo.yaml
```
To check the services:
```sh
kubectl get services
```
To check the services status :
```sh
kubectl get pods
```
Once all the status are on Running, we can verify :
```sh
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```
### 5. Open the application to outside traffic 
```sh
kubectl apply -f gateway-api/bookinfo-gateway.yaml
```
We change the service type to ClusteIP:
```sh
kubectl annotate gateway bookinfo-gateway networking.istio.io/service-type=ClusterIP --namespace=default
```
let's check :
```sh
kubectl get gateway
```
you should see :
```sh
$ kubectl get gateway
NAME               CLASS   ADDRESS                                            PROGRAMMED   AGE
bookinfo-gateway   istio   bookinfo-gateway-istio.default.svc.cluster.local   True         4m48s
```
### 6. Access the application
We will connect to the Bookinfo productpage service through the gateway you just provisioned. 
To access the gateway, we need to use the kubectl port-forward command:
```sh
kubectl port-forward svc/bookinfo-gateway-istio 8080:80
```
Open your browser and navigate to **http://localhost:8080/productpage** to view the Bookinfo application.

If you refresh the page, you should see the book reviews and ratings changing as the requests are distributed across the different versions of the reviews service.

### 7. View the dashboard
#### 1. Install Kiali and the other addons and wait for them to be deployed.
```sh
kubectl apply -f addons
kubectl rollout status deployment/kiali -n istio-system
```

#### 2. Acces the Kialy dashboard
```sh
istioctl dashboard kiali
```

### 8. Uninstall 
```sh
kubectl delete -f addons
istioctl uninstall -y --purge

kubectl delete namespace istio-system

kubectl label namespace default istio-injection-
```