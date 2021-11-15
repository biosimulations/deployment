# BioSimulations Deployment

| BioSimulations Dev  | BioSimulations Prod | Argo CD      | NATS Operator | Nginx Ingress | Prometheus   | Grafana      | Cert Manager |
| -------------       | -------------       |------------- |-------------  |-------------  |------------- |------------- |------------- |
| [![App Status](https://deployment.biosimulations.org/api/badge?name=biosimulations-dev&revision=true)](https://deployment.biosimulations.org/applications/biosimulations-dev)  | [![App Status](https://deployment.biosimulations.org/api/badge?name=biosimulations-prod&revision=true)](https://deployment.biosimulations.org/applications/biosimulations-prod)  | [![App Status](https://deployment.biosimulations.org/api/badge?name=argo-cd&revision=true)](https://deployment.biosimulations.org/applications/argo-cd)| [![App Status](https://deployment.biosimulations.org/api/badge?name=nats-operator&revision=true)](https://deployment.biosimulations.org/applications/nats-operator)| [![App Status](https://deployment.biosimulations.org/api/badge?name=nginx-ingress&revision=true)](https://deployment.biosimulations.org/applications/nginx-ingress)| [![App Status](https://deployment.biosimulations.org/api/badge?name=prometheus&revision=true)](https://deployment.biosimulations.org/applications/prometheus)|[![App Status](https://deployment.biosimulations.org/api/badge?name=grafana&revision=true)](https://deployment.biosimulations.org/applications/grafana)| [![App Status](https://deployment.biosimulations.org/api/badge?name=cert-manager&revision=true)](https://deployment.biosimulations.org/applications/cert-manager)|



## About
This [repository](https://github.com/biosimulations/deployment) contains the configuration for deploying the [BioSimulations](https://github.com/biosimulations/biosimulations) platform onto a Kubernetes cluster. The repository is organized to support a [GitOps](#gitops) workflow and continuous deployment using [ArgoCD](https://argoproj.github.io/argo-cd/). The deployment can be monitored and configured at https://deployment.biosimulations.org
### BioSimulations
### GitOps
Git Ops is an approch to cluster and deployment manangement that uses a git repository as the single source of truth for the state of the cluster and deployment. The kubernetes cluster contains an continious delivery application (in our case ArgoCD) which monitors the repository and applies any changes. This allows for strict versioning of deployments and easy reverts incase of any issues. Since all changes to the deployment must go through the git repository, there is no need to worry about changes to the cluster that may be lost if the cluster is recreated or changed. You can learn more about this approach [here](https://www.weave.works/technologies/gitops/)
## Structure 
The repository is structured as follows: 
### Cluster
The top level cluster folder contains the declaration of each app that is running on the cluster. This includes BioSimulations itself, but also other cluster-wide infrastructure, such as Nginx and Cert-Manager. The declarations for each app are located in the `argocd` folder, while the actual resrouces for each are contained in their own respective folders
### Base and Overlays
This contains the template deployment for BioSimulations. The `base` and `overlay` approach comes from the `kustomize` project. The `overlays` folder contains various instances of BioSimulations deployments, such as the dev and production environments. 
### Config
The config folder contains Kustomize template for the creation of config maps that can be loaded into the approriate application for each overlay. The `kustomization.yml` file in each overlay points to the appropriate config folder to use for that deployment 
### Secrets
The secrets for the application are set up identically to the config, but use the Kustomize `secret-generator` instead of the `config-map-generator`. The folder itself is a private git submodule which allows the deployment to be public without revealing the secrets.
### Hack
Contains files and config that are not deployed. This is a staging area for experiments, past approaches, and various resources that are relevant

## Deployment Instructions

### Prerequisites
Make sure you have [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) installed. If you are connecting to the BioSimulations cluster hosted on Google Cloud you will also need the [Google Cloud SDK](https://cloud.google.com/sdk/docs/quickstart-cli) installed.
### Connect to Kubernetes Cluster
This will vary based on your cluster provider. For the main BioSimulations cluster on Google Cloud, first ensure that you are added to the biosimulations organization. Then run the following command:

```
gcloud container clusters get-credentials biosimulations --zone us-central1-c --project biosimulations
```

This will automatically configure your kubectl context to the connect to cluster.
### Deploy ArgoCD (bootstrap)
The applications are automatically installed and managed using [ArgoCD](https://argoproj.github.io/argo-cd/). The following command will install ArgoCD on your cluster:

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

If you receive the following error (or similar):
```
Error from server (InternalError): error when applying patch:
{"metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"extensions/v1beta1\",\"kind\":\"Ingress\",\"metadata\":{\"annotations\":{\"cert-manager.io/cluster-issuer\":\"letsencrypt-prod\",\"kubernetes.io/ingress.class\":\"nginx\",\"kubernetes.io/tls-acme\":\"true\",\"nginx.ingress.kubernetes.io/backend-protocol\":\"HTTPS\",\"nginx.ingress.kubernetes.io/ssl-passthrough\":\"true\"},\"name\":\"argocd-server-ingress\",\"namespace\":\"argocd\"},\"spec\":{\"rules\":[{\"host\":\"deployment.biosimulations.org\",\"http\":{\"paths\":[{\"backend\":{\"serviceName\":\"argocd-server\",\"servicePort\":\"https\"},\"path\":\"/\"}]}}],\"tls\":[{\"hosts\":[\"deployment.biosimulations.org\"],\"secretName\":\"argocd-secret\"}]}}\n"},"labels":null},"spec":{"rules":[{"host":"deployment.biosimulations.org","http":{"paths":[{"backend":{"serviceName":"argocd-server","servicePort":"https"},"path":"/"}]}}]}}
to:
Resource: "extensions/v1beta1, Resource=ingresses", GroupVersionKind: "extensions/v1beta1, Kind=Ingress"
Name: "argocd-server-ingress", Namespace: "argocd"
for: "argo-ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1beta1/ingresses?timeout=10s": no endpoints available for service "ingress-nginx-controller-admission"
```
Run the following: 
```
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```
This removes the validation provided by the ingress-controller. Since the controller has not yet been deployed, the validation will always fail. We must be sure that the argo-ingress at `cluster/argo.argo-ingress' is valid. The creation of the nginx-ingress-controller will re-enable the validation automatically after it is deployed.
Then, retry the command:

```
kubectl apply -f cluster/argocd/
```
### Configure ArgoCD
Apply the ArgoCD secrets folder, and create the secrets. These secrets are used to 
1. authenticate with the this repository, and the secrets submodule repository
2. provide the client secret to enable the GitHub authentication
3. provide the admin secret to enable the admin user to login to the ArgoCD UI

Run the following command: 
```
kubectl apply -f secrets/argocd
```

### Login to ArgoCD
You should now be able to access the ArgoCD UI at https://deployment.biosimulations.org. When first deployed, it is possible that the SSL configuration is not yet ready. Your browser may prevent you from accessing the UI. If this is the case, determine how to disable HSTS on your browser and try again. 

The ArgoCD UI is configured to use GitHub authentication. To login, you will need to provide your GitHub username and password and be a member of the "deployment" team in the biosimulations organization.

It is also possible to login using the admin account in the event that the Github authentication is not yet working. To login as the admin user you will first need to enable the admin account. In the 'cluster/argocd/argo-cm.yaml' file, change the 'admin.enabled' value to 'true'. Then, run the following command to determine the admin password:

```
kubectl get secrets -n argocd argocd-initial-admin-secret --template={{.data.password}} | base64 --decode
```
It is highly recommended that you disable the admin account once you have verified that the GitHub authentication is working.


## Continuous Deployment
Once the setup is complete, any changes to the configurations will automoattically be applied to the cluster by ArgoCD. This can be monitored at https://deployment.biosimulations.org
