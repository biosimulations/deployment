# BioSimulations Deployment
## About
This [repository](https://github.com/biosimulations/deployment) contains the configuration for deploying the [BioSimulations](https://github.com/biosimulations/biosimulations) platform onto a Kubernetes cluster. The repository is organized to support a [GitOps](#gitops) workflow and continuous deployment using [ArgoCD](https://argoproj.github.io/argo-cd/). The deployment can be monitored and configured at https://deployment.biosimulations.org
### BioSimulations
### GitOps
### Structure 
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


