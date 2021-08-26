# BioSimulations Deployment
## About
### BioSimulations
### GitOps
### Strcture 
## Deployment Instructions

### Connect to Kubernetes Cluster

### Deploy ArgoCD (bootstrap)
Initial install: 

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
