# bookmark-k8s

Kustomize-based Kubernetes manifests for deploying bookmark related services and its supporting infrastructure.

This repo follows a GitOps model: changes to manifests are automatically picked up and deployed by ArgoCD in configured environments (e.g. QA, Prod).

## Deploying with ArgoCD

Ensure ArgoCD is installed in your cluster (via Helm or manifests). Then apply the app definition:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo/argo-cd -n argocd
kubectl apply -f apps/bookmark-api-qa.yaml -n argocd
```

ArgoCD will track the overlays/qa path and deploy any changes pushed to this repo.

### Auto-Sync

This repo assumes ArgoCD is set to automatically sync updates for QA. If you're using manual sync for Prod, use the CLI:

argocd app sync bookmark-api-prod

## PostgreSQL Setup (Bitnami Helm Chart)

Note: The following values are examples. You should update the username, password, and database name before deploying to a shared or production cluster.

This project uses PostgreSQL deployed inside the cluster using the Bitnami Helm chart.

### Install Steps

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

kubectl create namespace bookmark-db

helm install postgres bitnami/postgresql \
  --namespace bookmark-db \
  --set auth.username=bookmark \
  --set auth.password=bookmarkpass \
  --set auth.database=bookmark

### Connect inside cluster

kubectl run pg-client --rm -it --restart=Never --namespace bookmark-db \
  --image=bitnami/postgresql:latest \
  --env="PGPASSWORD=bookmarkpass" \
  --command -- psql -h postgres-postgresql -U bookmark -d bookmark

### Connection URL (for .env)

DATABASE_URL=postgresql+asyncpg://bookmark:bookmarkpass@postgres-postgresql.bookmark-db.svc.cluster.local:5432/bookmark

### Port Forward (for local access)

kubectl port-forward -n bookmark-db svc/postgres-postgresql 5432:5432

# Use this connection string instead:
DATABASE_URL=postgresql+asyncpg://bookmark:bookmarkpass@localhost:5432/bookmark
```

## Maintainer

@L2klbs
