# Service Mesh Demo with Istio

This guide demonstrates how to set up and deploy a simple service mesh using **Istio** within a Kubernetes cluster. The demo includes installing Istio, deploying a sample application, configuring a gateway, and monitoring the system using Kiali.

## Prerequisites
Ensure the following tools are installed before proceeding:
- [Minikube](https://minikube.sigs.k8s.io/docs/)
- [Istioctl](https://istio.io/latest/docs/setup/getting-started/)
- [Docker](https://www.docker.com/)
- [Kubernetes CLI (kubectl)](https://kubernetes.io/docs/tasks/tools/)

## Setup Instructions

### 1. Start Minikube
Start a local Kubernetes cluster with Minikube:
```sh
minikube start
```

### 2. Install Istio
Install Istio using the provided demo profile configuration:
```sh
istioctl install -f demo.yaml -y
```
Enable automatic sidecar injection for the `default` namespace:
```sh
kubectl label namespace default istio-injection=enabled
```

### 3. Install the Kubernetes Gateway API CRDs
Check if the required Custom Resource Definitions (CRDs) are present and install them if necessary:
```sh
kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
{ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.2.1" | kubectl apply -f -; }
```

### 4. Deploy the Demo Application
Deploy the **Bookinfo** sample application:
```sh
kubectl apply -f platform/kube/bookinfo.yaml
```
Verify the services are running:
```sh
kubectl get services
```
Check the status of the deployed pods:
```sh
kubectl get pods
```
Once all pods show a `Running` status, verify the application is accessible internally:
```sh
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```
> Expected output:
> ```<title>Simple Bookstore App</title>```

### 5. Open the Application to External Traffic
Apply the **Gateway API configuration** to expose the application:
```sh
kubectl apply -f gateway-api/bookinfo-gateway.yaml
```
Change the service type to `ClusterIP`:
```sh
kubectl annotate gateway bookinfo-gateway networking.istio.io/service-type=ClusterIP --namespace=default
```
Confirm the gateway is configured:
```sh
kubectl get gateway
```
> Expected output:
> ```sh
> NAME               CLASS   ADDRESS                                            PROGRAMMED   AGE
> bookinfo-gateway   istio   bookinfo-gateway-istio.default.svc.cluster.local   True         4m48s
> ```

### 6. Access the Application
Forward traffic from the Istio Gateway to your local machine:
```sh
kubectl port-forward svc/bookinfo-gateway-istio 8080:80
```
Now, open your browser and visit **http://localhost:8080/productpage**. You should see the Bookinfo application running.

If you refresh the page, different versions of the **reviews** service will be displayed as requests are distributed across multiple deployments.

### 7. Monitor the Application with Kiali
#### 1. Install Kiali and Additional Add-ons
Deploy Kiali and supporting components:
```sh
kubectl apply -f addons
kubectl rollout status deployment/kiali -n istio-system
```

#### 2. Access the Kiali Dashboard
Launch the Kiali UI for real-time service mesh monitoring:
```sh
istioctl dashboard kiali
```
This will open Kiali in your browser, providing insights into the service mesh topology and traffic flows.

### 8. Define the service versions
Run the following command to create default destination rules for the Bookinfo services:
```sh
kubectl apply -f networking/destination-rule-all.yaml
```

Wait a few seconds for the destination rules to propagate.

You can display the destination rules with the following command:
```sh
kubectl get destinationrules -o yaml
```


### Uninstall and Cleanup
To remove all deployed resources and uninstall Istio, run:
```sh
kubectl delete -f addons
istioctl uninstall -y --purge
kubectl delete namespace istio-system
kubectl label namespace default istio-injection-
```
This will ensure a clean environment, removing all Istio components and related configurations.
