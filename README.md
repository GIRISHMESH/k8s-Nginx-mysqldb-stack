
Nginx + MySQL Kubernetes Stack
Overview
This repository contains the Kubernetes manifests for deploying a production-ready Nginx web application with a MySQL backend, secured via TLS and managed through Nginx Ingress Controller.
Key features include:
•	Highly available MySQL StatefulSet with persistent storage (3 replicas).
•	Nginx web deployment behind a ClusterIP service.
•	Ingress Controller for HTTP/HTTPS routing with static public IP.
•	TLS support for secure communication.
•	Network policies to restrict access to MySQL.
•	Init jobs for database setup.
•	Resource requests & limits, readiness/liveness probes for reliability.

Architecture
       ┌─────────────────────┐
       │    Internet / DNS   │
       └─────────┬──────────┘
                 │
                 ▼
       ┌─────────────────────┐
       │ Nginx Ingress LB    │
       │  (LoadBalancer)     │
       └─────────┬──────────┘
                 │
                 ▼
       ┌─────────────────────┐
       │  Nginx Deployment   │
       │   ClusterIP Service │
       └─────────┬──────────┘
                 │
                 ▼
       ┌─────────────────────┐
       │   MySQL StatefulSet │
       │ Headless + ClusterIP│
       └─────────────────────┘
•	Namespaces:
o	nginx-stack: MySQL + Nginx app.
o	ingress-nginx: Ingress Controller.
•	Security:
o	MySQL credentials stored in Kubernetes secrets.
o	TLS for Ingress to encrypt traffic.
o	NetworkPolicy restricting MySQL access to Nginx pods.

Prerequisites
1.	Kubernetes Cluster (Azure AKS recommended).
2.	kubectl installed and configured to the cluster.
3.	Azure Static Public IP for the LoadBalancer:

   
az network public-ip create \
  --resource-group <NODE_RESOURCE_GROUP> \
  --name myStaticPublicIP \
  --sku Standard \
  --allocation-method static


   
5.	TLS certificates (base64-encoded) for your domain.

Deployment Instructions
1.	Create Namespaces, Secrets, and ConfigMaps:
kubectl apply -f 01-namespaces-secrets-configmaps.yaml
2.	Create StorageClass:
kubectl apply -f 02-storageclass.yaml
3.	Deploy MySQL StatefulSet & Services:
kubectl apply -f 03-mysql-statefulset-services.yaml
4.	Apply NetworkPolicy:
kubectl apply -f 04-networkpolicy.yaml
5.	Run MySQL Init Job (creates initial database/schema):
kubectl apply -f 05-mysql-init-job.yaml
6.	Deploy Nginx Application:
kubectl apply -f 06-nginx-deployment-service.yaml
7.	Deploy Nginx Ingress Controller:
kubectl apply -f 07-ingress-controller.yaml
8.	Create Ingress Resource (routes domain traffic to Nginx):
kubectl apply -f 08-ingress-resource.yaml

Validation & Access
•	Check MySQL Pods:
kubectl get pods -n nginx-stack -l app=mysql
•	Check Nginx Pods:
kubectl get pods -n nginx-stack -l app=nginx-web
•	Check Ingress Controller & LB IP:
kubectl get svc -n ingress-nginx
•	Access Application:
https://web.yourdomain.com

Security Considerations
•	Secrets Management: Credentials and TLS certs are stored in Kubernetes secrets. Consider integrating with Azure Key Vault for production-grade security.
•	Network Policy: Only Nginx pods can communicate with MySQL pods.
•	Non-root MySQL: MySQL runs as non-root (fsGroup: 999).
•	TLS: All external traffic is encrypted using Ingress TLS.

Scaling & HA
•	MySQL: 3 replicas with PersistentVolumeClaims for HA and data retention.
•	Nginx: 2 replicas behind ClusterIP service and Ingress LoadBalancer.
•	Ingress Controller: 2 replicas with LoadBalancer for high availability.

Notes
•	Replace <STATIC_IP> in the manifests with your Azure Public IP.
•	Replace web.yourdomain.com with your actual domain.
•	For production, consider external-dns integration to automatically manage DNS records.
•	Adjust resource requests/limits based on expected load.

Cleanup
To delete all resources:
kubectl delete namespace nginx-stack ingress-nginx
kubectl delete storageclass mysql-storage

This README provides full context, deployment order, security, and access instructions, making it easy for any DevOps engineer to deploy and maintain this stack.

