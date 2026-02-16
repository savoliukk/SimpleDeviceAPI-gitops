# SimpleDevice (GitOps)

This repo deploys **SimpleDevice API** and a **PostgreSQL** instance into two namespaces:

- `simpledevice-dev`
- `simpledevice-prod`

## Secrets policy (no secrets in Git)

This repo intentionally **does not store secret values**.
You must create the following Kubernetes Secrets in each namespace:

### 1) App secret: `simpledevice-api-secrets`
Must contain the connection string key expected by the API.

Recommended key:
- `ConnectionStrings__DefaultConnection`

Example:

```bash
# DEV
kubectl -n simpledevice-dev create secret generic simpledevice-api-secrets \
  --from-literal=ConnectionStrings__DefaultConnection='Host=postgres;Port=5432;Database=devices;Username=app;Password=<DEV_PASSWORD>' \
  --dry-run=client -o yaml | kubectl apply -f -

# PROD
kubectl -n simpledevice-prod create secret generic simpledevice-api-secrets \
  --from-literal=ConnectionStrings__DefaultConnection='Host=postgres;Port=5432;Database=devices;Username=app;Password=<PROD_PASSWORD>' \
  --dry-run=client -o yaml | kubectl apply -f -
```

### 2) Postgres secret: `postgres-secret`
Must contain:
- `postgres-password`

```bash
# DEV
kubectl -n simpledevice-dev create secret generic postgres-secret \
  --from-literal=postgres-password='<DEV_PASSWORD>' \
  --dry-run=client -o yaml | kubectl apply -f -

# PROD
kubectl -n simpledevice-prod create secret generic postgres-secret \
  --from-literal=postgres-password='<PROD_PASSWORD>' \
  --dry-run=client -o yaml | kubectl apply -f -
```

## Postgres persistence notes (kind / single node)

The manifests use `hostPath` PVs:

- `/data/postgres-dev`
- `/data/postgres-prod`

Create directories on the node if needed.
For a kind cluster (node is a Docker container):

```bash
docker exec -it simpledevice-control-plane mkdir -p /data/postgres-dev /data/postgres-prod
```

For production (EKS), replace this with a real StorageClass (EBS) or use AWS RDS.
