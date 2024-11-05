# CI/CD Pipeline for Jenkins with Kubernetes Deployment
### This repository contains a CI/CD pipeline setup using Jenkins, Docker, Kubernetes, SonarQube, Trivy, OWASP Dependency-Check, and Grafana. The pipeline automates code quality checks, security scans, Docker image builds, and Kubernetes deployments, while Grafana provides monitoring capabilities for Jenkins.

## Prerequisites
* **Docker and Docker Compose**: Installed on your server or local machine.
* **Kubernetes Cluster** : A running Kubernetes cluster with kubectl configured.
* **GitHub** : A GitHub repository for source code and configuration files.

## Pipeline Overview
This Jenkins pipeline is designed to automate the CI/CD process, from code quality checks to deployment on Kubernetes, with Grafana monitoring Jenkins health metrics. The key features of this pipeline include:

* **Source Code Management (SCM)**: Pulls code from GitHub.
* **SonarQube Quality Gate**: Checks code quality.
* **OWASP Dependency-Check**: Scans for vulnerabilities in dependencies.
* **Docker Build**: Builds a Docker image for the application.
* **Trivy Scan**: Scans Docker images for vulnerabilities.
* **Docker Push**: Pushes the image to Docker Hub.
* **Kubernetes Deployment**: Updates the Kubernetes deployment with the latest image.
* **Grafana Monitoring**: Monitors Jenkins metrics

## Pipeline Stages
The pipeline stages in Jenkins include:

1. **Clear Workspace**: Cleans the Jenkins workspace.

2. **Source Code Management (SCM)**: Pulls code from GitHub.

3. **SonarQube Quality Gate**: Runs SonarQube for code quality checks.

4. **OWASP Dependency-Check**: Scans for security vulnerabilities in dependencies.

5. **Building Docker Image**: Builds a Docker image tagged with the build number.

6. **Trivy Vulnerability Scan**: Scans the Docker image for vulnerabilities.

7. **Push to Docker Hub**: Pushes the Docker image to Docker Hub.

8. **Kubernetes Deployment with ArgoCD**:
  * **Manifest Repository Update**: After building and scanning the Docker image, the pipeline updates a separate [Kubernetes manifest](https://github.com/rrrrrrrjk/kubernetes-manifest) repository to reflect the new image tag. This repository contains YAML configuration files defining the Kubernetes resources (e.g., Deployments, Services) needed to run the application.
  * **ArgoCD Deployment**: ArgoCD monitors the manifest repository for changes. Once it detects the updated image tag, it automatically deploys the new version to the Kubernetes cluster. ArgoCD ensures a continuous deployment (CD) workflow, providing real-time synchronization and rollback options if needed.

9. **Clean Up**: Removes the Docker image from Jenkins to save space.
