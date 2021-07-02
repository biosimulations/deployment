# Deployment Instructions
## Connect to Kubernetes Cluster
## Deploy ArgoCD 
Initial install: 

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
