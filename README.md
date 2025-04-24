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
