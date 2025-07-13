# 🚀 Container Orchestration: Learner Report Application Deployment

This repository contains Kubernetes configuration and a CI/CD pipeline setup to deploy a **MERN (MongoDB, Express.js, React.js, Node.js)** application using **Kubernetes** and **Jenkins**.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Application Architecture](#application-architecture)
- [Local Setup](#local-setup)
- [Dockerization](#dockerization)
- [Kubernetes Deployment](#kubernetes-deployment)
  - [Database Deployment](#database-deployment)
  - [Backend Deployment](#backend-deployment)
  - [Frontend Deployment](#frontend-deployment)
- [CI/CD Pipeline with Jenkins](#cicd-pipeline-with-jenkins)
- [AWS EKS Deployment](#aws-eks-deployment)
- [Troubleshooting](#troubleshooting)

---

## 📘 Project Overview

This project demonstrates containerized deployment of a MERN stack application using container orchestration tools. The application consists of:
* **MongoDB** – Database
* **Node.js + Express.js** – Backend
* **React.js** – Frontend

---

## Prerequisites

- Docker
- Kubernetes (Minikube for local testing)
- kubectl
- AWS CLI (for EKS deployment)
- eksctl (for EKS deployment)
- Jenkins
- Git

## Application Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Frontend  │     │   Backend   │     │  Database   │
│  React.js   │────▶│  Express.js │────▶│   MongoDB   │
│   (Port 80) │     │ (Port 3001) │     │ (Port 27017)│
└─────────────┘     └─────────────┘     └─────────────┘
```

---

## Local Setup

### Clone Repositories
```bash
git clone https://github.com/UnpredictablePrashant/learnerReportCS_backend.git
git clone https://github.com/UnpredictablePrashant/learnerReportCS_frontend.git
```

### Setup Backend
```bash
cd learnerReportCS_backend
npm install
# Update config.env with MongoDB connection string
node index.js
```

### Setup Frontend
```bash
cd learnerReportCS_frontend
npm install --legacy-peer-deps
npm start
```

### Create User via Postman
Send a POST request to create a user before login:
- Endpoint: `http://localhost:3001/api/users/register`
- Method: POST
- Body: JSON with user credentials

---

## Dockerization

### Backend Docker Configuration
See [`Backend/Dockerfile`](./Backend/Dockerfile) for the Node.js backend containerization setup.

### Frontend Docker Configuration
- Production: See [`Frontend/Dockerfile`](./Frontend/Dockerfile) (multi-stage build with Nginx for Kubernetes)

### Build and Push Docker Images
```bash
# Backend
docker build -t learnerreport:backend .
docker tag learnerreport:backend yourusername/learnerreport:backend
docker push yourusername/learnerreport:backend

# Frontend (Production)
docker build -f Dockerfile.prod -t learnerreport:frontendforkubernetes .
docker tag learnerreport:frontendforkubernetes yourusername/learnerreport:frontendforkubernetes
docker push yourusername/learnerreport:frontendforkubernetes
```

---

## Kubernetes Deployment

All Kubernetes configuration files can be found in the [`Kubernetes Deployment`](./Kubernetes%20Deployment/) directory.

### Database Deployment

Apply the configurations:
```bash
kubectl apply -f ./Kubernetes\ Deployment/database/
```

### Backend Deployment

Apply the configurations:
```bash
kubectl apply -f ./Kubernetes\ Deployment/backend/
```

### Frontend Deployment

Apply the configurations:
```bash
kubectl apply -f ./Kubernetes\ Deployment/frontend/
```

---

## CI/CD Pipeline with Jenkins

The Jenkins pipeline configuration can be found in [`jenkins/Jekins-deployment.groovy`](./jenkins/Jekins-deployment.groovy).

### Setup Instructions
1. Create a new pipeline job in Jenkins
2. Configure it to use the Pipeline script from SCM
3. Set the SCM to Git and provide your repository URL
4. Set the script path to `Jenkinsfile`
5. Configure credentials for Docker Hub and AWS in Jenkins credentials store
6. Run the pipeline

---

## AWS EKS Deployment

### Create EKS Cluster
```bash
eksctl create cluster \
  --name manisha-eksctl-learnerapplication \
  --region us-west-1 \
  --version 1.31 \
  --nodegroup-name manisha-standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --with-oidc \
  --ssh-access \
  --ssh-public-key eks-key.pub \
  --managed
```

### Connect to EKS Cluster
```bash
aws eks --region us-west-1 update-kubeconfig --name manisha-eksctl-learnerapplication
```

### Deploy Application
Follow the same Kubernetes deployment steps as above, but make sure to update the frontend secret with the EKS backend service URL:

```bash
# Get backend service URL
kubectl get svc -n lrbackend

# Encode URL in base64
echo -n "http://BACKEND_SERVICE_URL:3001" | base64

# Update frontend secret with encoded URL
# Then apply all Kubernetes configurations
```

---

## Troubleshooting

### Frontend Not Connecting to Backend
- Verify the backend service is running: `kubectl get svc -n lrbackend`
- Check that the frontend secret contains the correct encoded backend URL
- Ensure all services are exposed correctly and have the right ports configured

### Database Connection Issues
- Check MongoDB pod status: `kubectl get pods -n lrdatabase`
- Verify the MongoDB connection string in the backend secret
- Ensure the PersistentVolume and PersistentVolumeClaim are correctly bound

### Jenkins Pipeline Failures
- Verify Docker Hub credentials are correctly configured in Jenkins
- Ensure AWS credentials are properly set up as Jenkins credentials
- Check that all path references in Jenkins pipeline script are correct
