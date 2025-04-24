# 🚀 Deploying Frappe on Kubernetes

Running **Frappe** on **Kubernetes (K8s)** enables scalable, resilient, and cloud-native deployments ideal for enterprise use cases. This guide provides step-by-step instructions to containerize and deploy Frappe using **Helm charts** and **Docker**.

---

## ✅ Prerequisites

Ensure you have the following tools and resources:

- A Kubernetes Cluster (e.g., Minikube, EKS, GKE, AKS)
- `kubectl` and `helm` installed
- Persistent Volume (PV) support (for MariaDB & ERPNext data)
- A domain and SSL certificate (optional, for production use)

---

## 🧱 Step 1: Set Up MariaDB Database

ERPNext requires MariaDB as its backend database. Deploy it with Helm:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install erpnext-db bitnami/mariadb \
  --namespace erpnext \
  --set auth.rootPassword=erpnextrootpass \
  --set auth.password=erpnextdbpass \
  --set persistence.enabled=true \
  --set persistence.size=10Gi
```
Verify that the database is running:

```bash
kubectl get pods -l app.kubernetes.io/name=mariadb
```

Explanation:
`kubectl get pods -l app.kubernetes.io/name=mariadb`: Lists all pods with the label app.kubernetes.io/name=mariadb to ensure the MariaDB database is running.
