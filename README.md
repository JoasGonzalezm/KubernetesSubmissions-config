# DevOps with Kubernetes — project configuration

Kubernetes configuration of the todo project. The application code lives in
[KubernetesSubmissions](https://github.com/JoasGonzalezm/KubernetesSubmissions); its GitHub
Actions build the images and commit the new image tags to this repository, and ArgoCD
deploys from here.

```text
apps/          manifests of each service (todo_app, todo_backend, todo_cronjob, todo_backup, broadcaster)
base/          what is common for all environments
overlays/
  staging/     namespace staging, every commit to main of the code repository
  production/  namespace production, every tag of the code repository, with database backups
argocd/        ArgoCD Applications for both environments
secrets/       SOPS encrypted secrets, applied outside of ArgoCD
```

## Setup

```sh
for NS in staging production; do
  kubectl create namespace $NS
  sops --decrypt secrets/todo-db-secret.enc.yaml | kubectl apply -n $NS -f -
done
kubectl create secret generic broadcaster-secret -n production --from-literal=WEBHOOK_URL=<url>
kubectl create secret generic storage-sa-key -n production --from-file=key.json=key.json
kubectl apply -f argocd/
```
