# ⚒️ Deploying Argocd on Kubernetes Cluster

Steps to deploy ArgoCD using helm chart on kubernetes cluster


## 🤳Pre-requisite for deploying ArgoCD

- [Kubernetes cluster] - AWS EKS / Azure AKS / on-prem cluster
- [helm installation] - Helm should be installed on laptop

### 💻 Steps for deploying ArgoCD
1. Create ArgoCD namespace
```sh
% kubectl create ns argocd
```

2. Add helm chart repo
```sh
% helm repo add argo https://argoproj.github.io/argo-helm
```

3. Search if ArgoCD helm chart is available in the repo
```sh
% helm search repo argo
NAME                      	CHART VERSION	APP VERSION  	DESCRIPTION                                       
argo/argo                 	1.0.0        	v2.12.5      	A Helm chart for Argo Workflows                   
argo/argo-cd              	5.22.1       	v2.6.2       	A Helm chart for Argo CD, a declarative, GitOps...
argo/argo-ci              	1.0.0        	v1.0.0-alpha2	A Helm chart for Argo-CI                          
argo/argo-events          	2.1.3        	v1.7.6       	A Helm chart for Argo Events, the event-driven ...
argo/argo-lite            	0.1.0        	             	Lighweight workflow engine for Kubernetes         
argo/argo-rollouts        	2.22.2       	v1.4.0       	A Helm chart for Argo Rollouts                    
argo/argo-workflows       	0.22.11      	v3.4.5       	A Helm chart for Argo Workflows                   
argo/argocd-applicationset	1.12.1       	v0.4.1       	A Helm chart for installing ArgoCD ApplicationSet 
argo/argocd-apps          	0.0.8        	             	A Helm chart for managing additional Argo CD Ap...
argo/argocd-image-updater 	0.8.4        	v0.12.2      	A Helm chart for Argo CD Image Updater, a tool ...
argo/argocd-notifications 	1.8.1        	v1.2.1       	A Helm chart for ArgoCD notifications, an add-o...

```

4. To install HA version of Argo-CD, make changes to values.yaml of helm chart and then pass that as a parameter.

Create a file ovverridden-values.yaml and override below values:

Configure redis in HA mode with persistence enabled -  So for AWS eks cluster underlying ebs volume will be created for each pod of redis statefulset.

Increase number of replicas for controller, server and repo-server to 3.

Disabled application-set and notifications for now.

```yaml
nameOverride: techslaves-ha

applicationSet:
  enabled: false

notifications:
  enabled: false

## Controller
controller:
  name: application-controller
  # If changing the number of replicas you must pass the number as ARGOCD_CONTROLLER_REPLICAS as an environment variable
  replicas: 3
  extraArgs:
  - --repo-server-timeout-seconds
  - "180"
  enableStatefulSet: true

# This key configures Redis-HA subchart and when enabled (redis-ha.enabled=true)
# the custom redis deployment is omitted
redis-ha:
  enabled: true
  persistentVolume:
    enabled: true

## Server
server:
  name: server
  replicas: 3
  extraArgs:
   - --insecure

## Repo Server
repoServer:
  name: repo-server
  replicas: 3
  env:
  - name: ARGOCD_EXEC_TIMEOUT
    value: 3m

```

5. Install HA version of ArgoCD
```sh
# helm install gitops-argocd argo/argo-cd -n argocd -f ovverridden-values.yaml
NAME: argocd
LAST DEPLOYED: Wed Mar  1 16:08:07 2023
NAMESPACE: argocd
STATUS: deployed
REVISION: 3
NOTES:
DEPRECATED option server.extraArgs."--insecure" - Use configs.params.server.insecure

In order to access the server UI you have the following options:

1. kubectl port-forward service/gitops-argocd-techslaves-ha-server -n argocd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)

```

5. Port forward to access ArgoCD UI from local
```sh
% kubectl port-forward service/gitops-argocd-techslaves-ha-server -n argocd 8080:443
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

6. Get admin password to login from ArgoCD UI
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
