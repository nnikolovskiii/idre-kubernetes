Here is a comprehensive documentation summary of your Kubernetes architecture. You can save this as `ARCHITECTURE.md` or `README_INFRA.md` in your repository for future collaborators.

---

# 🏗️ Infrastructure & Deployment Architecture

**Project:** LangGraph / GC Stack
**Cluster Provider:** Hetzner Cloud (K3s/K8s)
**Methodology:** GitOps (ArgoCD)

## 1. The Stack
| Component | Tool | Purpose |
| :--- | :--- | :--- |
| **Orchestration** | Kubernetes | Container management. |
| **Packaging** | Helm | Templating manifests (Deployments, Services, PVCs). |
| **CD / Deployment** | ArgoCD | Syncs cluster state with Git repositories. |
| **CI / Build** | GitHub Actions | Builds Docker images and updates Helm values. |
| **Metrics** | Prometheus & Grafana | Monitoring CPU, RAM, and Network traffic. |
| **Logs** | Loki & Promtail | Centralized logging and querying. |
| **Ingress** | Traefik | Handling incoming HTTP/HTTPS traffic. |

---

## 2. Repository Structure
We utilize a **Two-Repo GitOps Pattern** to separate source code from configuration.

### A. Application Repository (Source Code)
*   **Content:** Python/React code, Dockerfiles.
*   **Role:** Builds the artifacts.
*   **CI Pipeline:**
    1.  Builds Docker images (`frontend`, `backend`, `ai`).
    2.  Tags images with the Git Commit SHA.
    3.  Pushes images to Docker Hub.
    4.  **Trigger:** Checkouts the *Kubernetes Repository* and updates `values.yaml` with the new tags.

### B. Kubernetes Repository (Infrastructure)
*   **Content:** Helm Charts, ArgoCD Application manifests.
*   **Path:** `./charts/gc-app` (Main Application Chart).
*   **Role:** Defines the "Desired State" of the cluster.
*   **CD Pipeline:** Monitored by ArgoCD. When `values.yaml` changes here, the cluster updates automatically.

---

## 3. Deployment Workflow
**How to deploy a new feature:**
1.  Developer pushes code to `master` in the **Application Repo**.
2.  **GitHub Actions** automatically builds the image and edits `values.yaml` in the **Kubernetes Repo**.
3.  **ArgoCD** detects the change in the Kubernetes Repo (typically within 3 minutes).
4.  ArgoCD applies the new manifest to the cluster (Rolling Update).

**Manual Overrides:**
If you need to change a configuration (e.g., increase Memory Limit):
1.  Edit `charts/gc-app/values.yaml` in the Kubernetes Repo.
2.  Commit and Push.
3.  ArgoCD syncs automatically.

---

## 4. Observability (Monitoring & Logs)
We do not rely on `kubectl` for monitoring. We use the **Grafana Stack**.

### Accessing Dashboards
Since Ingress is not configured for internal tools (security), access via Port Forward:
```bash
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80
```
*   **URL:** `http://localhost:3000`
*   **Credentials:** `admin` / `admin` (or configured value).

### Usage
1.  **Metrics (CPU/RAM):** Go to **Dashboards** → **Kubernetes / Compute Resources / Namespace (Pods)**.
2.  **Logs:** Go to **Explore** → Select **Loki** → Run query: `{namespace="default"}`.

**Configuration Note:**
*   The Prometheus stack is installed via ArgoCD pointing to the official Helm chart.
*   *Critical Fix:* `ServerSideApply=true` is enabled in `argocd-prometheus.yaml` to handle large CRDs.
*   *Critical Fix:* Loki datasource is injected via `additionalDataSources` in `values` to bypass Grafana UI connection issues.

---

## 5. Secrets Management
Secrets are **not** stored in Git.
*   **File:** `.env.ai` (Local only, `.gitignore`d).
*   **Mechanism:** Secrets are injected manually into the cluster.
*   **Command to update secrets:**
    ```bash
    kubectl create secret generic app-env --from-env-file=.env.ai --dry-run=client -o yaml | kubectl apply -f -
    # Then restart pods to pick up changes
    kubectl rollout restart deployment backend ai
    ```

---

## 6. Directory Map (Kubernetes Repo)
```text
.
├── charts/
│   └── gc-app/           # Custom Helm Chart for the App
│       ├── templates/    # Deployment, Service, PVC, Ingress templates
│       ├── Chart.yaml    # Version info
│       └── values.yaml   # THE SOURCE OF TRUTH (Image tags, Config)
├── infra/                # Infrastructure configurations
│   ├── argocd-prometheus.yaml
│   └── argocd-loki.yaml
├── argocd-app.yaml       # The bridge connecting ArgoCD to this repo
└── README.md
```