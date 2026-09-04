# 8 — Argo CD app of apps

Step 7 plus one layer: a **root** Application whose "manifests" are Applications. You apply
one object; Argo CD applies everything else. To make the fan-out real there are now two
workloads: nginx from before, and [whoami](https://github.com/traefik/whoami), a tiny
server that echoes the request and its own pod name.

There is one root app per environment, and a cluster gets exactly one of them. In reality
staging and prod are different clusters; here you pick which one this cluster plays.

```
root-apps/staging.yaml  →  argocd-apps/overlays/staging  →  workloads/nginx/overlays/staging
        (you)                    (root app)             ↘  workloads/whoami/overlays/staging
                                                                 (child apps)
```

| Layer | Source | Applied by |
| --- | --- | --- |
| [`root-apps/`](root-apps) | — | you, `kubectl apply -f`, one of the two |
| the Applications | [`argocd-apps/`](argocd-apps) overlay: one Application per workload | Argo CD, from the root app |
| nginx, whoami | [`workloads/`](workloads), one kustomize tree each | Argo CD, from each child app |

| | staging | prod |
| --- | --- | --- |
| nginx | 1 replica, `staging.nginx-demo.locl`, ns `nginx-staging` | 3 replicas, `nginx-demo.locl`, ns `nginx-prod` |
| whoami | 1 replica, `whoami.staging.nginx-demo.locl`, ns `whoami-staging` | 2 replicas, `whoami.nginx-demo.locl`, ns `whoami-prod` |

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
for e in staging prod; do mkdir -p /tmp/nginx-demo/$e && cp -r workloads/nginx/overlays/$e/site /tmp/nginx-demo/$e/; done
```

Point `staging.nginx-demo.locl` and `whoami.staging.nginx-demo.locl` (for prod: `nginx-demo.locl`,
`whoami.nginx-demo.locl`) at your machine in your hosts file.

## Deploy

Staging shown; for prod use `root-apps/prod.yaml` and the prod hostnames throughout.

```bash
kubectl kustomize argocd-apps/overlays/staging    # render what the root app will build: two Applications
kubectl apply -f root-apps/staging.yaml
kubectl get applications -n argocd                # root-staging, nginx-demo-staging, whoami-staging — all Synced / Healthy
kubectl get pods -A -l 'app in (nginx-demo, whoami)'
curl -s http://staging.nginx-demo.locl/
curl -s http://whoami.staging.nginx-demo.locl/    # Hostname: is the pod name
```

## Clean up

```bash
kubectl delete -f root-apps/staging.yaml          # finalizers cascade: child apps, then their resources
kubectl get applications -n argocd                # empty
```

Then remove Argo CD itself:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```
