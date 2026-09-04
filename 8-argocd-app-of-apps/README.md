# 8 — Argo CD app of apps

Step 7 plus one layer: a **root** Application whose "manifests" are a step 7 Application.
You apply one object; Argo CD applies everything else.

There is one root app per environment, and a cluster gets exactly one of them. In reality
staging and prod are different clusters; here you pick which one this cluster plays.

```
root-apps/staging.yaml  →  argocd-apps/overlays/staging  →  workloads/overlays/staging
        (you)                    (root app)                      (child app)
```

| Layer                     | Source                                | Applied by                              |
|---------------------------|---------------------------------------|-----------------------------------------|
| [`root-apps/`](root-apps) | —                                     | you, `kubectl apply -f`, one of the two |
| the Application           | [`argocd-apps/`](argocd-apps) overlay | Argo CD, from the root app              |
| nginx                     | [`workloads/`](workloads) overlay     | Argo CD, from the child app             |

Argo CD itself stays outside the tree: it is installed by hand as before, not managed by
a root app.

## Precondition: Argo CD in the cluster

```bash
kubectl create namespace argocd
kubectl apply --server-side -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml   # server-side: a CRD is too big for client-side apply
kubectl apply --server-side -f argocd/             # plain-HTTP UI + ingress on argocd.nginx-demo.locl
kubectl -n argocd get pods # watch them starting
kubectl rollout restart -n argocd deployment/argocd-server
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

Point `argocd.nginx-demo.locl` at your machine in your hosts file, then log in at <http://argocd.nginx-demo.locl/>
as `admin` with that password.

## Setup

```bash
# Prepare files for the host volumes
for e in staging prod; do mkdir -p /tmp/nginx-demo/$e && cp -r workloads/overlays/$e/site /tmp/nginx-demo/$e/; done
```

Point `staging.nginx-demo.locl` (or `nginx-demo.locl` for prod) at your machine in your hosts file.

## Deploy

Staging shown; for prod use `root-apps/prod.yaml` and `nginx-demo.locl` throughout.

```bash
kubectl kustomize argocd-apps/overlays/staging    # render what the root app will build: one Application
kubectl apply -f root-apps/staging.yaml
kubectl get applications -n argocd                # root-staging and nginx-demo-staging, both Synced / Healthy
kubectl get pods -A -l app=nginx-demo
curl -s http://staging.nginx-demo.locl/
```

## Clean up

```bash
kubectl delete -f root-apps/staging.yaml          # finalizers cascade: child app, then nginx
kubectl get applications -n argocd                # empty
```

Then remove Argo CD itself:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```
