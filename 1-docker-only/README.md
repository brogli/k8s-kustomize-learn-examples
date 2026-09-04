# 1 — nginx with `podman run`

nginx serving [`site/index.html`](site/index.html) on <http://localhost:8080>.

```bash
podman run -d --name nginx-demo -p 8080:80 \
  -v "$PWD/site:/usr/share/nginx/html:ro" \
  docker.io/library/nginx:1.27-alpine
```

| Flag | Meaning |
| --- | --- |
| `-d` | run in the background |
| `--name nginx-demo` | container name |
| `-p 8080:80` | host port 8080 → container port 80 |
| `-v "$PWD/site:/usr/share/nginx/html:ro"` | mount `site/` over nginx's document root, read-only |

`docker` works the same way.

## Verify

```bash
curl -s http://localhost:8080
```

The page comes from the mount, not the image: edit `site/index.html` and reload.

## Clean up

```bash
podman rm -f nginx-demo
```

Next: [step 2](../2-docker-compose/README.md) — the same command as a Compose file.
