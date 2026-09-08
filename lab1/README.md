# Lab 1 - Kubernetes, Istio, and the Boutique Application

## Objective
Deploy the Boutique microservices application into a local Kubernetes cluster,
instrument it with Istio, and practice rolling updates by rebuilding the frontend image

## Environment
- OS: Windows 11 and WSL2
- Docker: 29.2.1
- kubectl: v1.34.1
- K8S context: docker-desktop

---

## Task 1 - Kubernetes and Istio Install

### What I did
- Enabled Kubernetes in Docker Desktop on single-node cluster
- Installed kubectl and set context to docker-desktop
- Installed istioctl and initialized Istio on the cluster
- Applied Gateway API CRDs
- Installed Istio addons: Prometheus, Grafana, Kiali

### Commands used
```bash
kubectl config use-context docker-desktop
kubectl get nodes
# istio install commands go here as you run them
```

### Result
```
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
grafana                1/1     1            1           7m41s
istio-egressgateway    1/1     1            1           11m
istio-ingressgateway   1/1     1            1           11m
istiod                 1/1     1            1           12m
jaeger                 1/1     1            1           7m41s
kiali                  1/1     1            1           7m41s
prometheus             1/1     1            1           7m40s
### Result
```
### Screenshot
`screenshots/Week1Task1.png`

---

## Task 2 - Deploy the Boutique Application

### What I did
- Cloned the microservices-demo repo from Google
- Applied K8S manifests using kubectl apply -k
- Port-forwarded frontend to localhost:8080

### Commands used
```bash
git clone https://github.com/googlecloudplatform/microservices-demo
cd microservices-demo
kubectl apply -k kustomize/
kubectl get pods
kubectl port-forward deployment/frontend 8080:8080
```

### Result
```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
adservice               1/1     1            1           39m
cartservice             1/1     1            1           39m
checkoutservice         1/1     1            1           39m
currencyservice         1/1     1            1           39m
emailservice            1/1     1            1           39m
frontend                1/1     1            1           39m
loadgenerator           1/1     1            1           39m
paymentservice          1/1     1            1           39m
productcatalogservice   1/1     1            1           39m
recommendationservice   1/1     1            1           39m
redis-cart              1/1     1            1           39m
shippingservice         1/1     1            1           39m
```
### Screenshot
`screenshots/Week1Task2a.png` - deployments
`screenshots/Week1Task2b.png` - Boutique homepage

---

## Task 3 - Rebuild Frontend Image and Deploy

### What I did
- Rebuilt the frontend Docker image locally
- Updated frontend.yaml to use local image
- Re-applied the deployment and verified rolling update

### Commands used
```bash
cd src/frontend
docker build -t frontend .
# I have edited kustomize/base/frontend.yaml
kubectl apply -f frontend.yaml
kubectl describe deploy/frontend
```

### Result
```
Name:                   frontend
Namespace:              default
CreationTimestamp:      Sun, 06 Sep 2026 23:23:58 +0200
Labels:                 app=frontend
Annotations:            deployment.kubernetes.io/revision: 4
Selector:               app=frontend
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app=frontend
  Annotations:      kubectl.kubernetes.io/restartedAt: 2026-09-07T00:00:32+02:00
                    sidecar.istio.io/rewriteAppHTTPProbers: true
  Service Account:  frontend
  Containers:
   server:
    Image:      frontend:latest
    Port:       8080/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     200m
      memory:  128Mi
    Requests:
      cpu:      100m
      memory:   64Mi
    Liveness:   http-get http://:8080/_healthz delay=10s timeout=1s period=10s #success=1 #failure=3
    Readiness:  http-get http://:8080/_healthz delay=10s timeout=1s period=10s #success=1 #failure=3
    Environment:
      PORT:                             8080
      PRODUCT_CATALOG_SERVICE_ADDR:     productcatalogservice:3550
      CURRENCY_SERVICE_ADDR:            currencyservice:7000
      CART_SERVICE_ADDR:                cartservice:7070
      RECOMMENDATION_SERVICE_ADDR:      recommendationservice:8080
      SHIPPING_SERVICE_ADDR:            shippingservice:50051
      CHECKOUT_SERVICE_ADDR:            checkoutservice:5050
      AD_SERVICE_ADDR:                  adservice:9555
      SHOPPING_ASSISTANT_SERVICE_ADDR:  shoppingassistantservice:80
      ENABLE_PROFILER:                  0
    Mounts:                             <none>
  Volumes:                              <none>
  Node-Selectors:                       <none>
  Tolerations:                          <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  frontend-754b6d944f (0/0 replicas created), frontend-66b5b57f65 (0/0 replicas created), frontend-78548db95 (0/0 replicas created)
NewReplicaSet:   frontend-867696c4d (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  59s   deployment-controller  Scaled up replica set frontend-867696c4d from 0 to 1
  Normal  ScalingReplicaSet  41s   deployment-controller  Scaled down replica set frontend-78548db95 from 1 to 0
```
### Screenshot
`screenshots/Week1Task3.png`

---

## Key Learnings
- Kubernetes orchestrates microservices, it maintain the working state, in case on pod failure or crashes K8s restart it, when there is high traffic it allocates neccessary pods as possible
- Istio sidecar injection must be enabled on the namespace first before the pods are started otherwise pods will be restarted to get Istio.
- 

## Issues Encountered
- The Istio sidecar injection did not got applied because the typo that i had in namespace label "istio-injection=enable" instead of "istio-injection=enabled" 
- I had to overwrite with ""kubectl label --overwrite with kubectl rollout restart deployment
