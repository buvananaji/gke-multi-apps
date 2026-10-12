# gke-multi-apps

Three sample apps deployed to GKE (`cluster123`) by one GitHub Actions workflow,
all served from **one public IP** through a GKE Ingress.

| App | Stack | URL path | Container port |
|---|---|---|---|
| `nginx-sample` | nginx static site | `/` | 80 |
| `hello-node` | Node.js (no deps) | `/node` | 8080 |
| `hello-python` | Python stdlib (no deps) | `/python` | 8080 |

```
apps/<name>/Dockerfile        image for the app
apps/<name>/k8s/              Deployment + ClusterIP Service (namespace: apps)
k8s/namespace.yaml            "apps" namespace
k8s/ingress.yaml              path routing → one Google HTTP load balancer
.github/workflows/deploy.yml  CI/CD
```

## How the pipeline works

- **Push to `main`** → only apps whose folder changed are built, pushed to
  `us-central1-docker.pkg.dev/test1-project-234813/docker-repo/<app>:<commit-sha>`
  and deployed (in parallel; one failing app doesn't cancel the others).
- Changes under `k8s/` re-apply the namespace and Ingress.
- **Actions → Deploy apps to GKE → Run workflow** deploys everything.

## One-time setup

1. Google Cloud keyless auth (Workload Identity Federation) for this repo — see
   the commands in `nginx-gke/README.md`, with `$REPO` set to this repository.
2. Repository secrets: `WIF_PROVIDER`, `WIF_SERVICE_ACCOUNT`.
3. Push to `main`, or run the workflow manually the first time.

## Find the public IP

```
kubectl -n apps get ingress apps
```

The Google load balancer takes ~5–10 minutes to become ready after the first deploy.

## Add another app

1. Create `apps/<new-app>/` with a `Dockerfile` and `k8s/` (copy an existing app,
   rename, keep `IMAGE_PLACEHOLDER`).
2. Add `<new-app>: apps/<new-app>/**` to the `filters:` in `deploy.yml`
   and to the `workflow_dispatch` list.
3. Add a path for it in `k8s/ingress.yaml`.
