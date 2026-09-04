# 2 — the same nginx with Compose

The step 1 command, written down as [`docker-compose.yml`](docker-compose.yml).

| Step 1 flag                           | Compose key                      |
|---------------------------------------|----------------------------------|
| `docker.io/library/nginx:1.27-alpine` | `image`                          |
| `--name nginx-demo`                   | `container_name`                 |
| `-p 8080:80`                          | `ports`                          |
| `-v "$PWD/site:…"`                    | `volumes` (relative to the file) |
| `-d`                                  | flag on `up`, not in the file    |

## Run

```bash
docker-compose up -d        # or: docker compose up -d
curl -s http://localhost:8080
docker-compose down
```

With Podman, Compose needs the Podman socket:

```bash
systemctl --user enable --now podman.socket
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
```

Next: [step 3](../3-k8s-manifests/README.md) — the same thing as Kubernetes manifests.
