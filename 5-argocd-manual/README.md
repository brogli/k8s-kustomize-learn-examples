# 5 — the same nginx, deployed by Argo CD

[`manifests/`](manifests) is step 3, unchanged. Nothing in it is applied by hand: Argo CD
pulls it from this repo on GitHub and applies it.

## Precondition: Argo CD in the cluster

```bash
kubectl create namespace argocd
kubectl apply --server-side -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml   # server-side: a CRD is too big for client-side apply
kubectl apply --server-side -f argocd/             # plain-HTTP UI + ingress on argocd.locl
kubectl -n argocd get pods # watch them starting
kubectl rollout restart -n argocd deployment/argocd-server
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

Point `argocd.nginx-demo.locl` at your machine in your hosts file, then log in at <http://argocd.nginx-demo.locl/>
as `admin` with that password.

## Setup

```bash
# Prepare files for the host volume
mkdir -p /tmp/nginx-demo && cp -r manifests/site /tmp/nginx-demo/
```

## Create the Application in the UI

**+ New App**, fill in, **Create**:

| Field            | Value                                                        |
|------------------|--------------------------------------------------------------|
| Application Name | `nginx-demo`                                                 |
| Project          | `default`                                                    |
| Sync Policy      | `Manual`                                                     |
| Repository URL   | `https://github.com/brogli/k8s-kustomize-learn-examples.git` |
| Revision         | `HEAD`                                                       |
| Path             | `5-argocd-manual/manifests`                                  |
| Cluster URL      | `https://kubernetes.default.svc`                             |
| Namespace        | `default`                                                    |

The app shows up **OutOfSync**: Argo CD has compared the repo with the cluster and found
the resources missing. **Sync** → **Synchronize** applies them.

```bash
kubectl get pods -l app=nginx-demo
curl -s http://nginx-demo.locl/
```

Now change something in the cluster by hand and watch Argo CD notice:

```bash
kubectl scale deployment/nginx-demo --replicas=1     # app goes OutOfSync; Sync puts it back to 2
```

## Clean up

In the UI: **Delete** the app (cascade on) — Argo CD removes what it created.

```bash
kubectl get pods -l app=nginx-demo                   # gone
```

Then remove Argo CD itself:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```
