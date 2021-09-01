# BioSimulations Deployment
## About
This repository contains the configuration for deploying the [BioSimulations](https://github.com/biosimulations/biosimulations) platfrom onto a kubernetes cluster. The repository is organized to support a [GitOps](#gitops) workflow and continious deployment using [ArgoCD](https://argoproj.github.io/argo-cd/). The deployment can me monitored and configured at https://deployment.biosimulations.org
### BioSimulations
### GitOps
### Structure 
## Deployment Instructions

### Connect to Kubernetes Cluster

### Deploy ArgoCD (bootstrap)
Initial install: 

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
