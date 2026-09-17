# KubeChat 🚀

KubeChat is a simple full-stack real-time chat application built with a React frontend, Node.js backend, and MongoDB database.

The main purpose of this project is to take a traditional full-stack application and containerize and deploy it using Docker and Kubernetes while implementing important Kubernetes concepts such as Deployments, Services, Persistent Storage, Secrets, and Ingress.

---

## 📌 Project Overview

KubeChat started as a simple full-stack chat application consisting of three main components:

- **Frontend** – React-based user interface
- **Backend** – Node.js/Express-based API and application logic
- **Database** – MongoDB for persistent application data

The application was first containerized using Docker. Separate Docker images were created for the frontend and backend, and the images were pushed to Docker Hub.

After containerization, the application was deployed on a Kubernetes cluster using manually created Kubernetes manifests.

The Kubernetes deployment includes separate workloads and Services for the frontend, backend, and MongoDB, along with persistent storage, Secrets, and Ingress-based routing.

---

## 🏗️ Architecture

```text
                         User
                          |
                          v
                  +----------------+
                  |     Ingress    |
                  |  NGINX Ingress |
                  +-------+--------+
                          |
             +------------+------------+
             |                         |
             v                         v
      +-------------+           +-------------+
      |  Frontend   |           |   Backend   |
      |   Service   |           |   Service   |
      +------+------+           +------+------+
             |                         |
             v                         v
      +-------------+           +-------------+
      |  Frontend   |           |   Backend   |
      |     Pod     |           |     Pod     |
      +-------------+           +------+------+
                                        |
                                        v
                                 +-------------+
                                 |   MongoDB   |
                                 |   Service   |
                                 +------+------+
                                        |
                                        v
                                 +-------------+
                                 |   MongoDB   |
                                 |     Pod     |
                                 +------+------+
                                        |
                                        v
                                  +-----------+
                                  | PVC / PV  |
                                  | Persistent|
                                  |  Storage  |
                                  +-----------+
🛠️ Technology Stack
Application
React
Node.js
Express.js
MongoDB
JavaScript
Containerization
Docker
Dockerfile
Docker Hub
Docker Compose
Kubernetes
Kubernetes
Minikube
kubectl
Deployments
Services
Namespace
PersistentVolume (PV)
PersistentVolumeClaim (PVC)
Secrets
Ingress
NGINX Ingress Controller
🐳 Dockerization

The first step was to containerize the application.

Separate Dockerfiles were created for the frontend and backend so that each application component could run independently inside its own container.

Frontend Image
afsarsohail/chatapp-frontend:latests
Backend Image
afsarsohail/chatapp-backend:latest

The Docker images were built locally and pushed to Docker Hub so that Kubernetes could pull and run them inside the cluster.

The MongoDB container uses the official MongoDB image instead of creating a custom MongoDB image.

☸️ Kubernetes Deployment

After successfully containerizing the application, the next step was deploying it on Kubernetes.

The Kubernetes configuration was created manually to understand how each Kubernetes resource works and how the different components communicate with each other.

1. Namespace

A dedicated namespace was created for the application:

chat-app

This keeps all KubeChat Kubernetes resources logically isolated from other workloads in the cluster.

2. Deployments

Separate Kubernetes Deployments were created for:

Frontend
Backend
MongoDB

Each Deployment manages the desired number of Pods for its respective application component.

Frontend Deployment

The frontend Deployment runs the Docker image created for the React application.

Backend Deployment

The backend Deployment runs the Node.js/Express application and exposes port 5001 inside the container.

MongoDB Deployment

The MongoDB Deployment uses the official MongoDB container image and runs MongoDB on port 27017.

3. Kubernetes Services

A separate Service was created for each component:

frontend
backend
mongodb
Frontend Service

Provides stable network access to the frontend Pods.

frontend:80
Backend Service

Provides stable internal access to the backend Pods.

backend:5001
MongoDB Service

Provides internal DNS-based communication between the backend and MongoDB.

mongodb:27017

The backend connects to MongoDB using the Kubernetes Service name rather than directly using the MongoDB Pod IP.

For example:

mongodb:27017

This allows Kubernetes to provide stable service discovery even if the MongoDB Pod is recreated.

💾 Persistent Storage

Since MongoDB stores application data, using only ephemeral Pod storage would risk losing data when the Pod is recreated.

To provide persistent storage, the project uses:

PersistentVolume (PV)
PersistentVolumeClaim (PVC)

The PVC requests storage from Kubernetes and becomes bound to an available PersistentVolume.

The MongoDB Pod mounts the persistent storage at:

/data/db

This allows MongoDB data to persist independently of the lifecycle of the MongoDB Pod.

MongoDB Pod
     |
     | mount
     v
   PVC
     |
     v
    PV
     |
     v
Persistent Storage
🔐 Kubernetes Secrets

Sensitive configuration such as the JWT secret and database credentials should not be hardcoded directly into application configuration.

Kubernetes Secrets are used to provide sensitive configuration to the backend application.

For example:

env:
  - name: JWT_SECRET
    valueFrom:
      secretKeyRef:
        name: backend-secret
        key: JWT_SECRET

This allows the application to consume sensitive values through environment variables without embedding them directly into the application code.

Note: Secrets should still be handled carefully. Kubernetes Secrets are not automatically equivalent to a dedicated external secret-management system.

🌐 Ingress

Instead of exposing the frontend and backend separately using different external ports, an NGINX Ingress Controller is used as the entry point for the application.

The Ingress routes incoming HTTP requests based on the request path.

Routing
chat-tws.com/
        |
        +----> frontend Service
        |
        +----> /api
                  |
                  +----> backend Service

The Ingress configuration contains two routes:

/      → frontend:80

/api   → backend:5001

This provides a single entry point for the application while allowing Kubernetes Services to remain responsible for internal service discovery and traffic routing.

🔄 Application Request Flow

The overall request flow is:

User
 |
 v
NGINX Ingress
 |
 +------ / ------> Frontend Service
 |                    |
 |                    v
 |                Frontend Pod
 |
 +---- /api ------> Backend Service
                      |
                      v
                  Backend Pod
                      |
                      v
                MongoDB Service
                      |
                      v
                  MongoDB Pod
                      |
                      v
                     PVC
📁 Kubernetes Configuration

The Kubernetes manifests are organized inside the K8S directory.

K8S/
│
├── namespace.yaml
│
├── frontend-deployment.yaml
├── frontend-service.yaml
│
├── backend-deployment.yaml
├── backend-service.yaml
│
├── mongodb-deployment.yaml
├── mongodb-service.yaml
│
├── mongo-pvc.yaml
│
├── backend-secret.yaml
│
└── ingress.yml
🚀 Deployment Process

The application was deployed in the following sequence:

1. Create Namespace
kubectl apply -f namespace.yaml
2. Deploy MongoDB
kubectl apply -f mongodb-deployment.yaml
kubectl apply -f mongodb-service.yaml
3. Create Persistent Storage
kubectl apply -f mongo-pvc.yaml
4. Create Backend Secret
kubectl apply -f backend-secret.yaml
5. Deploy Backend
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
6. Deploy Frontend
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
7. Configure Ingress
kubectl apply -f ingress.yml
🔍 Useful Kubernetes Commands

Check all Pods:

kubectl get pods -n chat-app

Check Services:

kubectl get svc -n chat-app

Check Deployments:

kubectl get deployments -n chat-app

Check PersistentVolumeClaims:

kubectl get pvc -n chat-app

Check Ingress:

kubectl get ingress -n chat-app

Describe Ingress:

kubectl describe ingress chatapp-ingress -n chat-app

Check Ingress Controller:

kubectl get pods -n ingress-nginx
🎯 What I Learned From This Project

This project provided practical experience with:

Containerizing a full-stack application using Docker
Creating Docker images using Dockerfiles
Pushing application images to Docker Hub
Creating and managing Kubernetes Namespaces
Creating Kubernetes Deployments
Running multiple application components as separate Pods
Exposing applications using Kubernetes Services
Understanding Kubernetes service discovery
Connecting a backend application with MongoDB through a Kubernetes Service
Using PersistentVolumes and PersistentVolumeClaims
Persisting MongoDB data using Kubernetes storage
Managing sensitive configuration using Kubernetes Secrets
Routing application traffic using NGINX Ingress
Understanding frontend, backend, and database communication inside Kubernetes
Deploying and troubleshooting a multi-component application on Minikube
🔮 Future Improvements

The project can be extended with additional Kubernetes and DevOps capabilities such as:

ConfigMaps for non-sensitive configuration
Liveness, Readiness and Startup Probes
CPU and Memory Requests/Limits
Horizontal Pod Autoscaler (HPA)
NetworkPolicies
ServiceAccounts and RBAC
Node Affinity
Taints and Tolerations
Helm
Argo CD
CI/CD using GitHub Actions or Jenkins
Container image vulnerability scanning
TLS/HTTPS with cert-manager
External secret management
Production Kubernetes deployment on AWS
👨‍💻 Project Goal

The goal of KubeChat is not only to build a chat application, but also to understand how a real multi-tier application can be containerized and deployed using Kubernetes.

The project demonstrates the transition from:

Local Application
       ↓
Docker Containers
       ↓
Docker Images
       ↓
Docker Hub
       ↓
Kubernetes
       ↓
Deployments
       ↓
Services
       ↓
Persistent Storage + Secrets
       ↓
Ingress
       ↓
Application

This project serves as a practical learning project for understanding containerization, Kubernetes networking, storage, configuration management, and application deployment.


### One thing I'd change before you commit this

Your README should **not expose actual passwords/JWT secrets**. Keep the Secret manifest out of the public repo if it contains real credentials, or replace values with placeholders such as:

```yaml
stringData:
  JWT_SECRET: "CHANGE_ME"

Also, because your actual MongoDB credentials differed from the original .env, don't document the real database password in the README.

This README is intentionally written around what you actually built—Docker → Docker Hub → Kubernetes Namespace → three Deployments/Services → MongoDB PV/PVC → Secrets → Ingress—rather than making it sound like a generic Kubernetes demo


![Application UI](<Screenshot 2026-09-17 204648.png>) ![alt text](<Screenshot 2026-09-17 201244.png>) ![Running k8s configuration](<Screenshot 2026-09-17 201507.png>)