# 🚀 Three-Tier Application CI/CD with Jenkins, SonarQube, Argo CD & Kubernetes

This project demonstrates a **production-grade CI/CD and GitOps pipeline** for a Spring Boot application using **Jenkins, Docker, SonarQube, Argo CD, and Kubernetes (Minikube)**.

The pipeline automates **build → code quality → containerization → GitOps-based deployment** with zero manual intervention.

---

## 🧱 Architecture Overview

**Flow:**

1. Developer pushes code to GitHub  
2. Jenkins pipeline is triggered  
3. Jenkins:
   - Builds the Spring Boot app using Maven
   - Runs SonarQube code quality analysis
   - Builds & pushes Docker image to Docker Hub
   - Updates Kubernetes manifest in Git (GitOps)
4. Argo CD detects manifest changes
5. Argo CD automatically deploys the app to Kubernetes (Minikube)

