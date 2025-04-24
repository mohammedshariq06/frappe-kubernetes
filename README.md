##🚀Introduction
Running Frappe on Kubernetes (K8s) provides scalability, high availability, and easy management for enterprise deployments. While Frappe is traditionally installed using Frappe Bench, containerizing it with Kubernetes ensures better resilience and cloud-native flexibility.
In this guide, we’ll walk through deploying Frappe on Kubernetes using Helm charts and Docker.
##✅Prerequisites
Before you begin, ensure you have the following:
 A Kubernetes Cluster (Minikube, EKS, GKE, or AKS)


 kubectl & Helm installed


 Persistent Volume (PV) support (for MariaDB & ERPNext data)


 Domain & SSL (optional, for production)
##🧱Step 1: Set Up a MariaDB Database
ERPNext requires MariaDB as its backend. Deploy it using Helm:
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install erpnext-db bitnami/mariadb --namespace erpnext --set auth.rootPassword=erpnextrootpass --set auth.password=erpnextdbpass --set persistence.enabled=true --set persistence.size=10Gi


Verify that the database is running:

kubectl get pods -l app.kubernetes.io/name=mariadb


Explanation:
kubectl get pods -l app.kubernetes.io/name=mariadb: Lists all pods with the label app.kubernetes.io/name=mariadb to ensure the MariaDB database is running.
##🧠Step 2: Deploy Redis for Caching
ERPNext uses Redis for real-time updates and caching. Deploy Redis with:
helm install erpnext-redis bitnami/redis --namespace erpnext --set auth.password=erpnextredispass --set persistence.enabled=true


##📦Step 3: Deploy Frappe
Now deploy ERPNext by linking it to the external MariaDB and Redis services:
helm install erpnext frappe/erpnext --set mariadb.enabled=false --set externalDatabase.host=erpnext-db-mariadb --set externalDatabase.user=root --set externalDatabase.password=erpnextrootpass --set redis.enabled=false --set externalRedis.host=erpnext-redis-master --set externalRedis.password=erpnextredispass --set persistence.enabled=true --set persistence.size=10Gi --set persistence.worker.storageClass=azurefile-csi


Explanation:
kubectl get pods: Verifies that all pods are running.


kubectl get pvc: Verifies that Persistent Volume Claims (PVCs) are bound.


kubectl get svc: Verifies the services are available.
##🌐Step 4: Expose Frappe via Ingress
To expose Frappe externally, set up an Ingress Controller (Nginx/Traefik) and apply the following Ingress resource:
Create erpnext-ingress.yaml:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: erpnext-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: erpnext.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: erpnext
            port:
              number: 80


Apply with:
kubectl apply -f erpnext-ingress.yaml


Explanation:
kubectl apply -f erpnext-ingress.yaml: Deploys the Ingress resource to expose Frappe externally via the specified domain.
##🛠️Step 5: Create MariaDB Config for Frappe
Create a custom MariaDB configuration for Frappe.
Create frappe-mariadb-cnf.yaml:

apiVersion: v1
kind: ConfigMap
metadata:
  name: frappe-mariadb-cnf
  labels: 
    app.kubernetes.io/component: primary
    app.kubernetes.io/instance: erpnext-db
    app.kubernetes.io/name: mariadb
data:
  my.cnf: |-
    [mysqld]
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci

    [mysql]
    default-character-set = utf8mb4




Upgrade MariaDB to use this configuration:

helm upgrade erpnext-db bitnami/mariadb --namespace erpnext --set auth.rootPassword=erpnextrootpass --set auth.password=erpnextdbpass --set persistence.enabled=true --set primary.existingConfigmap=frappe-mariadb-cnf


For more details, see the MariaDB configuration for Frappe.

Explanation:
The custom MariaDB configuration ensures compatibility with Frappe's required settings.
##🏗️Step 6: Create a New Site Template
Create a Kubernetes Job to create a new ERPNext site.
Create create_newsite.yaml:

apiVersion: batch/v1
kind: Job
metadata:
  name: create-erp-site
spec:
  backoffLimit: 0
  template:
    spec:
      securityContext:
        supplementalGroups: [1000]
      containers:
      - name: create-site
        image: frappe/erpnext-worker:v14
        args: ["bench","new-site","--db-host $(DB_HOST)","--db-port $(DB_PORT)","--db-root-username $(DB_ROOT_USER)","--db-root-password $(MYSQL_ROOT_PASSWORD)","--db-type mariadb","--admin-password $(ADMIN_PASSWORD)","$(SITE_NAME)"]
        imagePullPolicy: IfNotPresent
        volumeMounts:
          - name: sites-dir
            mountPath: /home/frappe/frappe-bench/sites
        env:
          - name: "SITE_NAME"
            value: erpkb15.metadottech.com
          - name: "DB_HOST"
            value: "erpnext-db-mariadb"
          - name: "DB_PORT"
            value: "3306"
          - name: "DB_ROOT_USER"
            value: root
          - name: "MYSQL_ROOT_PASSWORD"
            value: erpnextrootpass
          - name: "ADMIN_PASSWORD"
            value: adminerpnext456
      restartPolicy: Never
      volumes:
        - name: sites-dir
          persistentVolumeClaim:
            claimName: erpnext
            readOnly: false




Apply the job:

kubectl apply -f create_newsite.yaml


Explanation:
The create_newsite.yaml file defines a Kubernetes Job that creates a new ERPNext site using environment variables for configuration.

##🎯Step 7: Access ERPNext & Complete Setup
Get the LoadBalancer IP or domain name
Visit the URL: http://erpnext.yourdomain.com (or the LoadBalancer IP).


Complete the Frappe setup wizard (create admin user, configure site).
