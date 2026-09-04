# k8s + kustomize by example

This showcases the evolution of a simple nginx deployment from a docker|podman run command
up to a declarative app of apps pattern deployment with ArgoCD in k3s with Kustomize.

This way you can enter in the step you're familiar with and then learn how it translates further up.

| Step                                | Described as                                                            |
|-------------------------------------|-------------------------------------------------------------------------|
| [1](1-docker-only/README.md)        | a `podman run` command                                                  |
| [2](2-docker-compose/README.md)     | a Compose file                                                          |
| [3](3-k8s-manifests/README.md)      | Kubernetes manifests, `kubectl apply`                                   |
| [4](4-kustomize-overlays/README.md) | kustomize base + staging/prod overlays, `kubectl apply -k`              |
| [5](5-argocd-manual/README.md)      | the step 3 manifests, deployed by Argo CD from an app created in the UI |
| [6](6-argocd-declarative/README.md) | the same, with the Argo CD Application as a manifest                    |
| [7](7-argocd-kustomize/README.md)   | kustomize for the workload and for the Argo CD Applications             |
| [8](8-argocd-app-of-apps/README.md) | app of apps: one root Application deploys the Applications              |

Everything runs on your local machine: Podman (or Docker) and Compose for steps 1–2,
[k3s](https://k3s.io/) for step 3 onward.
