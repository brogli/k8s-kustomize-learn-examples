# k8s + kustomize by example

This showcases the evolution of a simple nginx deployment from a simple docker|podman run command
up to an app-of-apps deployment with argocd in k3s with kustomize.

| Step                                | Described as                                               |
|-------------------------------------|------------------------------------------------------------|
| [1](1-docker-only/README.md)        | a `podman run` command                                     |
| [2](2-docker-compose/README.md)     | a Compose file                                             |
| [3](3-k8s-manifests/README.md)      | Kubernetes manifests, `kubectl apply`                      |
| [4](4-kustomize-overlays/README.md) | kustomize base + staging/prod overlays, `kubectl apply -k` |

Everything runs on your local machine: Podman (or Docker) and Compose for steps 1–2,
[k3s](https://k3s.io/) for step 3 onward.

More steps follow.
