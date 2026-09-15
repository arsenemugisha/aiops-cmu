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



---

## Week 2 Task 1 — Monitor Boutique with Kiali

### What I did
- Opened Kiali dashboard using istioctl dashboard kiali
- Navigated to Workloads → shippingservice → Traffic tab
- Set Reported from to Source
- Observed healthy state showing both frontend and checkoutservice traffic

### Result
- frontend: 0.59rps, 100% success
- checkoutservice: 0.24rps, 100% success

### Screenshot
`screenshots/Week2Task1.png`

---

## Week 2 Task 2 — Fault Injection 500ms delay (Checkoutservice only)

### What I did
- Created shipping-delay.yaml VirtualService targeting checkoutservice → shippingservice
- Applied 500ms fixed delay at 100% for checkoutservice traffic only
- Frontend traffic to shippingservice was not affected

### Commands used
```bash
kubectl apply -f ~/aiops-cmu/lab1/configs/shipping-delay.yaml
```

### Result
- checkoutservice request duration spiked to ~500-600ms
- frontend request duration remained low and unaffected

### Screenshot
`screenshots/Week2Task2.png`

---

## Week 2 Task 3 — Fault Injection on one pod only (Checkoutservice scaled to 2)

### What I did
- Scaled checkoutservice to 2 pods
- Added label delay=true to one pod using kubectl edit pod
- Created new VirtualService matching only the labeled pod with 1sec delay
- Observed ~500ms average in Kiali — one pod delayed, one not

### Commands used
```bash
kubectl scale deploy checkoutservice --replicas=2
kubectl edit pod <podID>
kubectl apply -f ~/aiops-cmu/lab1/configs/checkout-delay-pod.yaml
```

### Result
- Kiali showed flat 500ms average for checkoutservice over several minutes
- Confirms one pod at 1sec delay + one pod at 0ms = ~500ms average

### Screenshot
`screenshots/Week2Task3.png`

---

## Key Learnings
- Kubernetes orchestrates microservices by maintaining desired state where crashed pods are restarted automatically
- Istio sidecar injection must be enabled on the namespace BEFORE pods are deployed
- 2/2 in kubectl get pods means the app container plus the Istio envoy sidecar proxy
- Rolling updates allow zero-downtime deployments in Kubernetes
- VirtualService is the Istio object that controls traffic routing and fault injection
- Fault injection can target specific traffic sources using sourceLabels without changing application code
- Pod labels can be used to target individual pods within the same deployment for selective fault injection
- Prometheus, Grafana, Kiali, Loki and Jaeger form a complete observability stack

## Issues Encountered
- Istio sidecar injection failed initially due to a typo in the namespace label on istio-insjection=enable instead of istio-injection=enabled. Fixed with kubectl label --overwrite and kubectl rollout restart deployment
- Pods showed 1/1 instead of 2/2 because the label typo prevented Istio from injecting the envoy sidecar
- PATH variable lost between WSL2 sessions — fixed permanently by adding istio bin to ~/.bashrc