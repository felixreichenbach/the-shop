# The Shop

A simple 3-tier online shop for Oktoberfest booth items. See [REQUIREMENTS.md](REQUIREMENTS.md) for the full requirements.

Run the whole stack with Docker:

```shell
docker compose up --build
```

- Frontend: http://localhost:8080
- Backend API: http://localhost:8000
- Demo login: `demo` / `demo123`

## Deploying to Kubernetes / minikube

Manifests are in `k8s/` (plain YAML + a `kustomization.yaml`, no Helm). They expect the container images to already exist in the cluster's image store as `oktoberfest-backend:latest` and `oktoberfest-frontend:latest`.

```shell
minikube start

# Build the images directly inside minikube (no registry needed)
minikube image build -t oktoberfest-backend:latest ./backend
minikube image build -t oktoberfest-frontend:latest ./frontend

kubectl apply -k k8s/

minikube service frontend -n oktoberfest   # opens the shop in your browser
```

Notes:

- There's no Compose-style `depends_on` in Kubernetes, so the backend retries its database connection internally with backoff on startup instead of relying on restarts — it comes up clean even if `db` isn't ready yet.
- `k8s/secret.yaml` ships the same demo credentials as `docker-compose.yml`, for the same reason — replace them before any real deployment.
- To tear down: `kubectl delete -k k8s/` (and `minikube stop` if you're done with the cluster).

## Running backend tests

The backend has a pytest suite (`backend/tests/`) covering auth, products, cart, and orders against an in-memory database — no Docker or Postgres needed. Run it before deploying any backend change:

```shell
cd backend
python3 -m venv .venv && .venv/bin/pip install -r requirements-dev.txt
.venv/bin/pytest -v
```
