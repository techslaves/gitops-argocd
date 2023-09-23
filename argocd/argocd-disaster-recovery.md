# ⚒️ ArgoCD disaster recovery

It is important to have a disaster recovery mechanism on devops infra for an organisation. ArgoCD disaster recovery is a way to restore argocd state, applications, settings, deployment history to another argocd instance.
This could also be helpful in blue-green deployment where we team is switching from one instance to other and wanted to retain the argocd state.

## Prerequisite
- ArgoCD should be installed on kubernetes cluster
- kubectl utility should be installed on local machine


## 🤳Steps to backup ArgoCD comfiguration running on kubernetes cluster (Manual):
- ArgoCD cli is preinstalled in argocd-application=controller pod which is used to take argocd backup.Exec into argocd-application-controller pod and run
```
% argocd admin export > argocd-backup.yaml
```
- copy backup from pod to local machine
```
% kubectl cp argocd/argocd-application-controller-0:/home/argocd/argocdbackup.yaml <path-of-local-laptop or ec2 machine>/argocd-backup.yaml
```

### 💻 Steps for restore ArgoCD on kubernetes cluster
1. Copy backup from local (mac) to argocd-application-controller pod
```sh
% kubectl cp argocd-backup.yaml <argocd-namespace>/<argocd-controller-pod-name>:/home/argocd
```
2. Execute below command to restore argocd state
```sh
% argocd admin import - < argocd-backup.yaml 
```

## 🤳Strategy to backup ArgoCD comfiguration running on Production kubernetes cluster (Automated):
To take backup from ArgoCD cluster running on kubernetes cluster create a cronJob which runs periodically maybe once a day, takes backup from ArgoCD and store in S3 bucket. If required backup yaml file from S3 bucket can be used to restore argocd state.
