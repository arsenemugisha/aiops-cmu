# Lab 1 — Kubernetes, Istio, and the Boutique Application

## Objective
Deploy the Boutique microservices application into a local Kubernetes cluster,
instrument it with Istio, and practice rolling updates by rebuilding the frontend image.

## Environment
- OS: Windows 11 + WSL2
- Docker: 29.2.1
- kubectl: v1.34.1
- K8S context: docker-desktop

---

## Task 1 — Kubernetes and Istio Install

### What I did
- Enabled Kubernetes in Docker Desktop (single-node cluster)
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
<!-- kubectl get deploy -n istio-system output here -->
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
grafana                1/1     1            1           7m41s
istio-egressgateway    1/1     1            1           11m
istio-ingressgateway   1/1     1            1           11m
istiod                 1/1     1            1           12m
jaeger                 1/1     1            1           7m41s
kiali                  1/1     1            1           7m41s
prometheus             1/1     1            1           7m40s

### Screenshot
`screenshots/Week1Task1.png`

---

## Task 2 — Deploy the Boutique Application

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
<!-- paste kubectl get deploy output here -->

### Screenshot
`screenshots/Week1Task2a.png` — deployments
`screenshots/Week1Task2b.png` — Boutique homepage

---

## Task 3 — Rebuild Frontend Image and Deploy

### What I did
- Rebuilt the frontend Docker image locally
- Updated frontend.yaml to use local image
- Re-applied the deployment and verified rolling update

### Commands used
```bash
cd src/frontend
docker build -t frontend .
# edited kustomize/base/frontend.yaml
kubectl apply -f frontend.yaml
kubectl describe deploy/frontend
```

### Result
<!-- paste kubectl describe deploy/frontend output here -->

### Screenshot
`screenshots/Week1Task3.png`

---

## Key Learnings
- 
- 
- 

## Issues Encountered
- 
