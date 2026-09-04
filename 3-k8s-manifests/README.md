# 3 — the same nginx as Kubernetes manifests

| File                                 | Object     | Role                                                                                                             |
|--------------------------------------|------------|------------------------------------------------------------------------------------------------------------------|
| [`deployment.yaml`](deployment.yaml) | Deployment | the container: image, replicas, `/tmp/nginx-demo/site` mounted at `/usr/share/nginx/html` as a `hostPath` volume |
| [`service.yaml`](service.yaml)       | Service    | a stable address in front of the Pods                                                                            |
| [`ingress.yaml`](ingress.yaml)       | Ingress    | routes `http://localhost/` to the Service, via the Traefik that ships with k3s                                   |

## Setup

```bash
curl -sfL https://get.k3s.io | sh -                 # installs k3s and kubectl
mkdir -p ~/.kube                                     # so kubectl works without sudo
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown "$USER" ~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc   # k3s's kubectl ignores ~/.kube/config otherwise
source ~/.bashrc
```

A `hostPath` volume is a directory on the node — with k3s, your machine. Put `site/`
where the Deployment expects it:

```bash
# Prepare files for the host volume
mkdir -p /tmp/nginx-demo && cp -r site /tmp/nginx-demo/
```

## Deploy

```bash
kubectl apply -f .
kubectl get pods,ingress -l app=nginx-demo
curl -s http://localhost/
```

If a Pod is not `Running`:

```bash
kubectl describe pod <name>        # events at the bottom
kubectl logs <name>
```

## Clean up

```bash
kubectl delete -f .
```

Next: [step 4](../4-kustomize-overlays/README.md) — the same manifests, with a staging and a prod overlay.
