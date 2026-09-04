# 6 — the same nginx, with a declarative Argo CD Application

Step 5 again, but the "New App" form is now a manifest: [`application.yaml`](application.yaml).
[`manifests/`](manifests) is still step 3, unchanged.

| UI field (step 5)              | In the manifest                                                   |
|--------------------------------|-------------------------------------------------------------------|
| Application Name               | `metadata.name`                                                   |
| Project                        | `spec.project`                                                    |
| Repository URL, Revision, Path | `spec.source`                                                     |
| Cluster URL, Namespace         | `spec.destination`                                                |
| Sync Policy                    | `spec.syncPolicy` — automated this time, with self-heal and prune |

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
# Prepare files for the host volume
mkdir -p /tmp/nginx-demo && cp -r manifests/site /tmp/nginx-demo/
```

## Deploy

The only thing applied by hand is the ArgoCD-Application. Argo CD does the rest:

```bash
kubectl apply -f application.yaml
kubectl get applications -n argocd        # SYNC STATUS Synced, HEALTH STATUS Healthy
kubectl get pods -l app=nginx-demo
curl -s http://nginx-demo.locl/
```

Self-heal in action:

```bash
kubectl scale deployment/nginx-demo --replicas=1
kubectl get pods -l app=nginx-demo -w     # back to 2 within seconds, without a Sync click
```

## Clean up

```bash
kubectl delete -f application.yaml        # the finalizer makes Argo CD remove the nginx resources first
```

Then remove Argo CD itself:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```

Next: [step 7](../7-argocd-kustomize/README.md) — kustomize for the workload and for the Applications.
