# Argo CD 

This folder contains the Argo CD configuration files. Applying this folder will bootstrap Argo CD and the rest of the applications in the cluster. 


The kustomization file includes the following:
- The stable branch of Argo CD high availability installation manifests.
- The stable branch of the image updater plugin.
- The ingress definition for the deployment ui.
- The repository connection information for the deployment repo. 
- The cluster secrets
- The "App of Apps" for the rest of the applications in the cluster.

The kustomization file includes the following modifications:
- The config map for the Argo CD configuration.
- The RBAC configuration for the Argo CD deployment.
- The list of known hosts for the Argo CD deployment. 

