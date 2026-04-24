# ArgoCD ApplicationSet — Bootstrap

## Prerequisites

- A running Kubernetes cluster
- `kubectl` configured
- `helm` CLI installed

---

## 1. Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for pods:
```bash
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=120s
```

Get the initial admin password:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath='{.data.password}' | base64 -d && echo
```

Access the UI (port-forward for local testing):
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# UI → https://localhost:8080  user: admin
```

---

## 2. Push this repo to GitHub

1. Create a new GitHub repo (e.g. `YOUR_ORG/argo-app`)
2. Update the `repoURL` in [applicationsets/apps-appset.yaml](applicationsets/apps-appset.yaml):
   ```yaml
   repoURL: https://github.com/YOUR_ORG/argo-app.git
   ```
3. Push:
   ```bash
   git init
   git remote add origin https://github.com/YOUR_ORG/argo-app.git
   git add .
   git commit -m "initial gitops setup"
   git push -u origin main
   ```

If the repo is **private**, register it in ArgoCD:
```bash
argocd repo add https://github.com/YOUR_ORG/argo-app.git \
  --username YOUR_USER --password YOUR_TOKEN
```

---

## 3. Apply the ApplicationSet

```bash
kubectl apply -f applicationsets/apps-appset.yaml
```

This creates **6 Applications** automatically: `dev-nginx`, `dev-apache`, `staging-nginx`, `staging-apache`, `prod-nginx`, `prod-apache`.

Check them:
```bash
kubectl get applications -n argocd
```

---

## 4. To test with only one environment

Comment out the `staging` and `prod` entries in the Matrix `list` generator inside [applicationsets/apps-appset.yaml](applicationsets/apps-appset.yaml).

---

## Repo Structure

```
argo-app/
├── applicationsets/
│   └── apps-appset.yaml        # The ApplicationSet definition
├── envs/
│   ├── dev/
│   │   ├── nginx/values.yaml
│   │   └── apache/values.yaml
│   ├── staging/
│   │   ├── nginx/values.yaml
│   │   └── apache/values.yaml
│   └── prod/
│       ├── nginx/values.yaml
│       └── apache/values.yaml
└── bootstrap/
    └── README.md               # This file
```

## Charts Used (Docker Hub OCI)

| App    | Chart ref                                    | Version |
|--------|----------------------------------------------|---------|
| nginx  | `registry-1.docker.io/bitnamicharts/nginx`   | 18.3.5  |
| apache | `registry-1.docker.io/bitnamicharts/apache`  | 11.3.4  |
