[README.md](https://github.com/user-attachments/files/31180676/README.md)
# 2-Tier Flask + MySQL on Kubernetes

A two-tier web application deployed on Kubernetes, separating the **application layer** (Flask)
from the **data layer** (MySQL). The app tier is stateless and horizontally scalable; the data
tier uses persistent storage so data survives pod restarts. A Helm chart is included for
parameterized, repeatable deployments.

## 🏗️ Architecture

```
                ┌─────────────────────┐
   Client  ───▶ │   Flask Service      │
                │   (flask-svc.yml)    │
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │  Flask Deployment     │
                │  (flask-dep.yml)      │
                │  — application tier   │
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │   MySQL Service       │
                │   (mysql-svc.yml)     │
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │  MySQL Deployment     │
                │  — data tier          │
                └──────────┬───────────┘
                           │
                ┌──────────▼───────────┐
                │  PersistentVolume +   │
                │  PersistentVolumeClaim│
                │  (sql-pv.yml,         │
                │   mysql-pvc.yml)      │
                └───────────────────────┘
```

- **Application tier** — Flask app, deployed and exposed via `flask-dep.yml` / `flask-svc.yml`.
- **Data tier** — MySQL, deployed and exposed via its own deployment / `mysql-svc.yml`, backed by
  a `PersistentVolume` (`sql-pv.yml`) and `PersistentVolumeClaim` (`mysql-pvc.yml`) so data
  persists independently of the pod lifecycle.
- **Helm chart** — the `helm-2-tier/` directory packages the same architecture as a parameterized
  chart for repeatable, environment-configurable installs.

## 📁 Project Structure

```
2-tier-flask-sql-app/
├── helm-2-tier/        # Helm chart version of the deployment
├── deployment.yml       # Deployment configuration
├── flask-dep.yml         # Flask application Deployment
├── flask-svc.yml          # Flask application Service
├── mysql-pvc.yml            # MySQL PersistentVolumeClaim
├── mysql-svc.yml              # MySQL Service
├── sql-pv.yml                   # PersistentVolume for MySQL data
└── syl-dep.yml                    # MySQL Deployment
```

## ⚙️ Prerequisites

- A running Kubernetes cluster (e.g. Minikube, Kind, or a cloud-managed cluster)
- `kubectl` configured to point at your cluster
- [Helm](https://helm.sh/) (optional — only required if deploying via the Helm chart)

## 🚀 Deploying

### Option 1 — Raw manifests

Apply the PersistentVolume and PersistentVolumeClaim first so the data tier has storage ready
before MySQL starts, then apply the rest:

```bash
kubectl apply -f sql-pv.yml
kubectl apply -f mysql-pvc.yml
kubectl apply -f syl-dep.yml
kubectl apply -f mysql-svc.yml
kubectl apply -f flask-dep.yml
kubectl apply -f flask-svc.yml
kubectl apply -f deployment.yml
```

Or apply everything at once:

```bash
kubectl apply -f .
```

### Option 2 — Helm chart

```bash
helm install two-tier-app ./helm-2-tier
```

### Verify the deployment

```bash
kubectl get pods
kubectl get svc
kubectl get pvc
```

## 🧹 Tearing down

```bash
kubectl delete -f .
# or, if installed via Helm:
helm uninstall two-tier-app
```

## 🔭 Future Improvements

- [ ] Add Prometheus + Grafana manifests for real-time monitoring and dashboards
- [ ] Add a NetworkPolicy to restrict traffic between tiers
- [ ] Add resource requests/limits to each Deployment
- [ ] Add a CI/CD pipeline (e.g. GitHub Actions or Jenkins) for automated builds and deploys
- [ ] Add a proper `README` diagram export and architecture decision notes

## 🛠️ Tech Stack

**Application:** Flask (Python) · **Database:** MySQL · **Orchestration:** Kubernetes, Helm
