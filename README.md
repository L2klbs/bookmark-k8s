# bookmark-k8s

This repo contains the Kubernetes manifests for deploying the bookmark-api and bookmark-ui services. It uses Kustomize to manage per-environment configuration and supports GitOps-style deployment.

### Structure
```basn
bookmark-k8s/
├── base/
│   ├── bookmark-api/
│   └── bookmark-ui/
├── overlays/
│   ├── qa/
│   │   ├── bookmark-api/
│   │   └── bookmark-ui/
│   ├── stage/
│   ├── prod/
```

### Local Testing with Minikube

```bash
# Start Minikube and enable tunneling:
minikube start
minikube tunnel
```

Update /etc/hosts with your local domains:

```bash
# /etc/hosts
127.0.0.1 api.bookmark.local
127.0.0.1 ui.bookmark.local
```

Deploy the QA environment:

```bash
kubectl apply -k overlays/qa/
```

Your API should be available at: http://api.bookmark.local</br>
Your UI should be available at: http://ui.bookmark.local

### Services and Ports

bookmark-api: listens on port 8000 (internally), exposed through a ClusterIP service on port 80

bookmark-ui: assumed to be served via a web server on port 80

### Ingress

Ingress is configured with NGINX to route external traffic to internal services based on subdomain:

api.bookmark.local → bookmark-api

ui.bookmark.local → bookmark-ui

### Secrets

Secrets are used to configure environment-specific values like DATABASE_URL. These are stored in Kubernetes as base64-encoded values and injected into the app at runtime.

Helm Dependencies (see bookmark-infra)

Some components (like Ingress and TLS) are installed via Helm in a separate repo: `bookmark-infra`