# 7 — Argo CD + kustomize

Two kustomize trees, one per thing that gets applied:

| Tree                          | What                                                      | Applied by              |
|-------------------------------|-----------------------------------------------------------|-------------------------|
| [`workloads/`](workloads)     | the nginx workload — step 4: base + staging/prod overlays | Argo CD                 |
| [`argocd-apps/`](argocd-apps) | step 6's Application — base + staging/prod overlays       | you, `kubectl apply -k` |

Argo CD runs `kustomize build` itself when `spec.source.path` holds a `kustomization.yaml`,
so each Application just points at a workload overlay. The Application overlays only
change three things:

|                              | staging                      | prod                      |
|------------------------------|------------------------------|---------------------------|
| `nameSuffix`                 | `nginx-demo-staging`         | `nginx-demo-prod`         |
| `spec.source.path`           | `workloads/overlays/staging` | `workloads/overlays/prod` |
| `spec.destination.namespace` | `nginx-staging`              | `nginx-prod`              |

## Precondition: Argo CD in the cluster

```bash
kubectl create namespace argocd
kubectl apply --server-side -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml   # server-side: a CRD is too big for client-side apply
kubectl apply --server-side -f argocd/             # plain-HTTP UI + ingress on argocd.k8s-demo.locl
kubectl -n argocd get pods # watch them starting
kubectl rollout restart -n argocd deployment/argocd-server
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

Point `argocd.k8s-demo.locl` at your machine in your hosts file, then log in at <http://argocd.k8s-demo.locl/>
as `admin` with that password.

## Setup

```bash
# Prepare files for the host volumes
for e in staging prod; do mkdir -p /tmp/nginx-demo/$e && cp -r workloads/overlays/$e/site /tmp/nginx-demo/$e/; done
```

Point `nginx.staging.k8s-demo.locl` and `nginx.prod.k8s-demo.locl` at your machine in your hosts file.

## Deploy

```bash
kubectl kustomize workloads/overlays/staging      # render the workload — what Argo CD will build
kubectl kustomize argocd-apps/overlays/staging    # render the Application — what apply sends
kubectl apply -k argocd-apps/overlays/staging
kubectl apply -k argocd-apps/overlays/prod
kubectl get applications -n argocd                # two apps, Synced / Healthy
kubectl get pods -A -l app=nginx-demo             # 1 + 3 pods
curl -s http://nginx.staging.k8s-demo.locl/
curl -s http://nginx.prod.k8s-demo.locl/
```

## Clean up

```bash
kubectl delete -k argocd-apps/overlays/staging
kubectl delete -k argocd-apps/overlays/prod
```

Then remove Argo CD itself:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```

Next: [step 8](../8-argocd-app-of-apps/README.md) — app of apps: a root Application deploys the Applications.
