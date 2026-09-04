# 4 — staging and prod with kustomize

The step 3 manifests, unchanged, in [`base/`](base). Each directory in
[`overlays/`](overlays) is a `kustomization.yaml` that layers one environment's
differences on top:

|              | staging                   | prod                 |
|--------------|---------------------------|----------------------|
| namespace    | `nginx-staging`           | `nginx-prod`         |
| replicas     | 1                         | 3                    |
| ingress host | `staging.nginx-demo.locl` | `nginx-demo.locl`    |
| content      | `overlays/staging/site`   | `overlays/prod/site` |
| memory limit | 64Mi (base)               | 128Mi                |

How the overlay expresses that, in [`overlays/staging/kustomization.yaml`](overlays/staging/kustomization.yaml):

| Field       | Does                                                                            |
|-------------|---------------------------------------------------------------------------------|
| `resources` | what to build on: the base, plus a `Namespace`                                  |
| `namespace` | sets `metadata.namespace` on everything                                         |
| `replicas`  | overrides the Deployment's replica count                                        |
| `patches`   | partial YAML merged into a resource, or a JSON patch aimed at one with `target` |

## Setup

Same k3s as step 3. Put each overlay's `site/` where its Deployment expects it, and point
both hostnames at your machine in your hosts file:

```bash
# Prepare files for the host volumes
for e in staging prod; do mkdir -p /tmp/nginx-demo/$e && cp -r overlays/$e/site /tmp/nginx-demo/$e/; done
```

## Deploy

```bash
kubectl kustomize overlays/staging     # render — see what apply would send
kubectl apply -k overlays/staging
kubectl apply -k overlays/prod
kubectl get pods -A -l app=nginx-demo
curl -s http://staging.nginx-demo.locl/
curl -s http://nginx-demo.locl/
```

## Clean up

```bash
kubectl delete -k overlays/staging
kubectl delete -k overlays/prod
```

Next: [step 5](../5-argocd-manual/README.md) — the step 3 manifests, deployed by Argo CD.
