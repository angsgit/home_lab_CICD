# 🚀 Home Lab - CI/CD, Kubernetes & Monitoring

## ⚡️ TL;DR — End-to-End AWS DevOps Pipeline

This project demonstrates a **complete automated DevOps workflow on AWS**, built entirely using Infrastructure as Code (IaC) principles.

![DEPLOYED APP](documentation/screenshots/homelab.jpg)
---

### 🚀 Pipeline Flow

1. **Infrastructure as Code (IaC)**
   - Terraform provisions AWS EC2 instances, networking, and security groups.
   - Ansible installs and configures Kubernetes (master + worker nodes).
   - Managed via the `Jenkinsfile-infra` pipeline, with a `DESTROY_INFRA` flag for teardown.

2. **Source Control**
   - Developer pushes code to **GitHub** (Flask app + Jenkinsfiles + manifests).
   - A **webhook** automatically triggers the Jenkins CI pipeline.

3. **Continuous Integration (CI)**
   - Jenkins pulls code and runs unit tests.
   - Builds a Docker image for the Flask app.
   - Pushes the image to **Docker Hub**.

4. **Continuous Deployment (CD)**
   - Jenkins triggers the **app-deploy-aws** pipeline.
   - Ansible applies Kubernetes manifests to the cluster (namespace, Deployment, Service, and Ingress).
   - Waits for rollout completion and prints the application access URL.

5. **Monitoring & Observability**
   - **Prometheus** scrapes metrics from Kubernetes and the Flask app.
   - **Grafana** visualizes system and app performance dashboards.
   - Monitoring stack deployed via Helm for automation.

6. **Disaster Recovery (DR)**
   - Multi-region AWS support using Terraform workspaces and region variables.
   - **Velero** backs up and restores Kubernetes resources to S3.
   - **EBS snapshots** replicate persistent storage across regions.
   - **Route 53 failover** automatically routes traffic to the DR cluster if the primary fails.
   - Jenkins pipelines allow manual or scheduled DR drills.

---

### 🧠 Key Outcomes
- Fully automated **infrastructure + application pipeline** from code → AWS → Kubernetes.  
- Zero manual configuration: everything is reproducible via Jenkins and code.  
- Integrated **monitoring**, **disaster recovery**, and **multi-region failover** for production resilience.

---

**Result:**  
A complete, cloud-native DevOps ecosystem that automatically provisions infrastructure, deploys applications, and ensures high availability — all driven by code.

---

## 📘 Overview

This project demonstrates a complete **DevOps CI/CD pipeline** using a simple **Flask-based web application** deployed on a **Kubernetes cluster**.  

It covers key DevOps lifecycle components — **source control, build automation, containerization, orchestration, monitoring, and observability** — all running locally within a **fully functional home lab environment**.

The setup also emphasizes **High Availability (HA)**, **Disaster Recovery (DR)**, and **Service Reliability**, reflecting real-world enterprise **DevOps** and **Site Reliability Engineering (SRE)** practices.  

In future phases, the architecture will expand to include **multi-zone / multi-region cloud deployment** and **safe deployment strategies** such as **Blue-Green** and **Canary releases** to ensure zero-downtime updates.

---


## 🧩 Project Architecture

## 🧰 Tech Stack & Tools Used

| Category | Tool | Purpose |
|-----------|------|----------|
| **Source Control** | GitHub | Hosts application and IaC repositories |
| **CI/CD Automation** | Jenkins | Automates build, test, and deploy pipeline |
| **Containerization** | Docker | Packages the Flask app and dependencies |
| **Orchestration** | Kubernetes | Deploys and manages application containers |
| **Configuration Management** | Ansible | Automates environment setup across VMs |
| **Infrastructure as Code (IaC)** | Terraform |  Automates VM and cluster provisioning |
| **Code Quality & Scanning** | SonarQube | Static code analysis integrated into CI |
| **Artifact Registry** | DockerHub | Stores and versions container images |
| **Monitoring & Metrics** | Prometheus | Collects metrics from Kubernetes and applications |
| **Visualization & Dashboards** | Grafana | Visualizes performance and reliability metrics |
| **Reverse Proxy & Load Balancing** | NGINX  | Provides ingress and routing for web traffic |
| **Programming Language** | Python | Simple web app used for demonstration |

---


## 🏗️ High-Level Architecture & CI/CD Flow

This setup represents a **real-world DevOps pipeline** — from code commit to deployment and monitoring — running entirely inside a **AWS Kubernetes cluster**.


<pre>         
           │   Developer    │
           │   (Git Push)   │
           └──────┬─────────┘
                  │
                  ▼
        ┌────────────────────────────┐
        │        GitHub Repo         │
        │ (Source + Jenkinsfiles +   │
        │  Terraform + Ansible Code) │
        └──────────┬────────────────┘
                   │ Webhook Trigger
                   ▼
        ┌────────────────────────────┐
        │          Jenkins           │
        │     CI/CD Orchestrator     │
        ├────────────────────────────┤
        │                            │
        │ 1️⃣ Infra Pipeline (IaC)     │
        │  ─────────────────────────  │
        │  • Terraform → AWS Infra    │
        │  • Ansible → K8s Cluster    │
        │  • Triggers App Pipeline    │
        │                            │
        │ 2️⃣ App Deployment Pipeline │
        │  ─────────────────────────  │
        │  • Build Docker Image       │
        │  • Run Unit Tests           │
        │  • Push to Docker Hub       │
        │  • Deploy via Ansible + K8s │
        └──────────┬────────────────┘
                   │
                   ▼
        ┌─────────────────────┐
        │     Docker Hub      │
        │ (Container Images)  │
        └──────── ┬───────────┘
                  │
                  ▼
        ┌─────────────────────┐
        │   Kubernetes (K8s)  │
        │  (App Deployment)   │
        └────────┬────────────┘
                 │
                 ▼
        ┌─────────────────────┐
        │   Prometheus +      │
        │     Grafana         │
        │ (Monitoring Stack)  │
        └─────────────────────┘
 ``` </pre>
---

## ☁️ Phase 0 - Infrastructure as Code (IaC)

Before deploying any applications, the foundation infrastructure was fully automated using **Terraform** and **Ansible**, executed through a **Jenkins pipeline**.


### ✅ Provision AWS Resources with Terraform

- Creates EC2 instances for master and worker nodes.
- Configures networking, security groups, and SSH access.
- Outputs public IPs for dynamic inventory generation.


### ⚙️ Configure Kubernetes with Ansible

- Automatically installs and initializes Kubernetes on the master node.
- Joins worker nodes to the cluster.
- Sets up required namespaces, metrics server, and ingress controller.

### 🚀 Jenkins Automation

- The Jenkinsfile-infra pipeline handles both Terraform and Ansible execution.
- Includes a DESTROY_INFRA parameter to optionally tear down AWS resources.
- When successful, it triggers the app deployment pipeline automatically.


--- 

## 🏗️ Phase 1 - Flask App Setup

- Built a simple Python Flask application with `/` and `/metrics` endpoints.
- Added a `requirements.txt` file for dependencies.
- Verified the app runs locally using:
  ```bash
  python main.py
  ```
- Confirmed `/metrics` exposes Prometheus-style metrics for monitoring.

---


## 🐳 Phase 2 - Dockerization

- Created a `Dockerfile`:
  ```dockerfile
  FROM python:3.10-slim
  WORKDIR /app
  COPY requirements.txt .
  RUN pip install --no-cache-dir -r requirements.txt
  COPY . .
  EXPOSE 8000
  CMD ["python", "main.py"]
  ```

- Built and tested the container:
  ```bash
  docker build -t flask-app .
  docker run -p 8000:8000 flask-app
  ```

---


## ☸️ Phase 3 - Kubernetes Deployment

- Deployed the Flask app to Kubernetes with a **Deployment**, **Service**, and **Ingress**.
- Example manifest:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: webapp
    namespace: app-v1
  spec:
    replicas: 4
  
  ```
  **K8S Running Pods:**

  ![DEPLOYED APP](documentation/screenshots/k8s.jpg)

 **Verified the app via: http://homelab:30788**

![DEPLOYED APP](documentation/screenshots/app.jpg)


---


## 🔁 Phase 4 - CI/CD with Jenkins

- Configured **Jenkins** with:
  - Git plugin
  - Docker plugin
  - Kubernetes CLI
  - Credentials for Docker Hub and kubeconfig
- Jenkins automatically:
  1. Pulls code from GitHub on push  
  2. Builds the Docker image  
  3. Pushes it to Docker Hub  
  4. Updates the Kubernetes deployment with the new image tag  
  5. Rolls out the new version automatically

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        KUBECONFIG_CREDENTIALS = credentials('kubeconfig-file')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/angsgit/devops_project.git'
            }
```

**Jenkins Pipeline:**

![DEPLOYED APP](documentation/screenshots/jenkins.jpg)

---


## 📊 Phase 5 - Monitoring & Observability

- Installed **Helm**:
  ```bash
  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  ```
- Added the Prometheus community repo and installed the monitoring stack:
  ```bash
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts  
  helm install kube-prometheus prometheus-community/kube-prometheus-stack -n monitoring
  ```
- Exposed Grafana to the host:
  ```bash
  kubectl -n monitoring patch svc kube-prometheus-grafana -p '{"spec": {"type": "NodePort"}}'
  kubectl -n monitoring get svc kube-prometheus-grafana
  ```
  → Accessed Grafana via `http://<master-node-ip>:<nodeport>`

**Prometheus scraping Metrics:**

![DEPLOYED APP](documentation/screenshots/promth.jpg)


**Grafana Visualising Metrics:**

![DEPLOYED APP](documentation/screenshots/grafana.jpg)


**Custom Dashboard in Grafana for APP Metrics (HTTP Total Requests):**

![DEPLOYED APP](documentation/screenshots/grafana_app.jpg)

---

## ☁️ Phase 6 - Disaster Recovery (DR)

To ensure high availability and resilience, a Disaster Recovery (DR) strategy was integrated into the AWS DevOps pipeline.  
All components — infrastructure, configuration, and applications — are reproducible using Infrastructure as Code (IaC).

---

### 🧩 Multi-Region Infrastructure-as-Code

- The Terraform configuration supports **multi-region deployment** by parameterizing the AWS region.
- Jenkins can deploy to either the **primary** or **secondary (DR)** AWS region.

### We create two workspaces or two Jenkins pipeline parameters:

#### Primary region: eu-west-1
#### DR region: eu-central-1

variable "aws_region" {
  default = "eu-west-1"  # Primary region
}

provider "aws" {
  region = var.aws_region
}

- Deploy to the DR region:
terraform apply -var="aws_region=eu-central-1"

#### This way, you can spin up an identical Kubernetes cluster in another AWS region on demand.

---
### 🪄 Automated Backup & Restore

For application and configuration state:

- Use Velero in Kubernetes:

- Backs up namespaces, deployments, and services to S3.
- Can restore them in another region or clusster.

  velero backup create full-cluster-backup --include-namespaces app-v1
  velero restore create --from-backup full-cluster-backup

- For persistent data (if using databases, volumes, etc.):
    - Use EBS snapshots or AWS Backup.
    - Replicate snapshots to another region automatically using AWS Backup policies.
---

### 📦 Docker Image Redundancy

- Push images to Amazon ECR (Elastic Container Registry) or Docker Hub.
- Enable cross-region replication in ECR:
    - Ensures container images are available in both primary and DR regions instantly.
---

### 🔁 Jenkins Multi-Region DR Automation

  - Add a parameter in Jenkins:

    parameters {
        choice(name: 'AWS_REGION', choices: ['eu-west-1', 'eu-central-1'], description: 'Select region to deploy infrastructure')
    }

  - Run the same pipeline to deploy infra and apps to your DR region.
  - Optionally, schedule weekly DR rehearsals (using Jenkins cron triggers).

---

###  🌐 DNS-Level Failover (Route 53)

- Use Route 53 failover routing:

- Primary cluster: health-checked endpoint.

  DR cluster: secondary endpoint (activated automatically if primary fails).

  Example:
  app.mydomain.com
  ├── Primary → eu-west-1 load balancer
  └── Secondary → eu-central-1 load balancer

###  📊 Monitoring & Alerts

  - Use Prometheus + Grafana stack to monitor both regions.
  - Add AWS CloudWatch alarms:
  - Detect EC2 instance/ELB/Kubernetes API unavailability.
  - Trigger Jenkins or Lambda for auto-failover deployment.

###  ✅ DR Summary
  - Component	DR Strategy
  - Infrastructure	Terraform deploys to secondary region
  - Kubernetes state	Velero backups to S3
  - App data	EBS snapshot replication
  - Docker images	ECR cross-region replication
  - DNS failover	Route 53 health checks
  - CI/CD continuity	Jenkins parameterized multi-region deploy
  - Monitoring	Prometheus + Grafana + CloudWatch


## 🧠 Key Learnings

- End-to-End DevOps Pipeline: Built a complete CI/CD workflow — from source code in GitHub to application deployment on Kubernetes.
- Infrastructure as Code (IaC): Automated AWS infrastructure provisioning using Terraform, followed by Ansible for Kubernetes setup and configuration.
- Jenkins Automation: Used Jenkins pipelines triggered by Git webhooks to orchestrate both infrastructure creation and application deployment.
- Containerization & Deployment: Dockerized a Flask application, pushed the image to Docker Hub, and deployed it to Kubernetes clusters using Ansible playbooks.
- Kubernetes Management: Gained experience managing manifests, namespaces, services, and ingress controllers in Kubernetes.
- Monitoring & Observability: Deployed Prometheus and Grafana (via Helm charts) for real-time metrics collection and visualization.
- Cluster Operations: Learned to handle node taints, resource requests/limits, and pod scheduling for better reliability and resource utilization.

---

## 🗺️ Next Steps (Planned)

- Integrate Alertmanager notifications (Slack or email) 
- Deploy to a cloud-managed Kubernetes cluster (EKS / AKS)

---

## 📂 Repository Structure

```
.
project/
│
├── README.md                           # Full project documentation and setup guide
├── .gitignore                          # Ignore unnecessary files (logs, creds, etc.)
│
├── app/                                # Flask application source code
│   ├── main.py                         # Flask entry point
│   ├── requirements.txt                # Python dependencies
│   ├── Dockerfile                      # Container build instructions
│
├── IaC/                                # Infrastructure as Code (Terraform + Ansible)
│   ├── terraform/                      # Terraform for AWS infrastructure
│   │   ├── main.tf                     # EC2 instances, security groups, networking
│   │   ├── variables.tf                # Input variables
│   │   ├── outputs.tf                  # Exported IPs for Ansible inventory
│   │   ├── provider.tf                 # AWS provider and backend configuration
│   │   └── terraform.tfstate           # Terraform state file (generated)
│   │
│   ├── ansible/                        # Ansible playbooks for configuration & deployment
│   │   ├── install_kubernetes.yml      # Installs Kubernetes on AWS EC2 instances
│   │   ├── deploy_k8s_app.yml          # Deploys Flask app to Kubernetes cluster
│   │   ├── inventory.ini               # Dynamic inventory generated from Terraform outputs
│   │   └── roles/                      # (Optional) Future modularization for Ansible roles
│   │
│   └── Jenkinsfile-infra               # Jenkins pipeline for provisioning & setup
│
├── jenkins/                            # Jenkins CI/CD pipeline configuration
│   ├── Jenkinsfile-app-deploy          # Pipeline for build → push → deploy (App)
│
├── k8s/                                # Kubernetes manifests and Helm resources
│   ├── webappv1.yaml                   # Deployment, Service, and Ingress for the Flask app
│   ├── namespace.yaml                  # Custom namespace definition (e.g. app-v1)
│   ├── helm/                           # Helm charts for monitoring stack
│   │   ├── prometheus/
│   │   └── grafana/
│
├── monitoring/                         # Monitoring stack (Helm-based)
│   ├── prometheus/                     # Prometheus custom configurations
│   ├── grafana/                        # Grafana dashboards and custom configs
│   │   ├── dashboards/
│   │   └── datasources/
│
└── documentation/                      # Documentation and visuals
    ├── architecture_diagram.png        # Jenkins → Terraform → Ansible → K8s pipeline
    ├── screenshots/                    # Grafana dashboards, Jenkins UI, Flask app views
    │   ├── grafana-dashboard.png
    │   ├── jenkins-pipeline.png
    │   └── flask-app.png
    └── notes.md                        # Summary notes, troubleshooting logs, future plans


```

---

## 🔒 Security

All sensitive credentials and configuration files (e.g. Docker Hub tokens, kubeconfig, Grafana admin passwords) are **not stored in this repository**.  
They are securely managed using **Jenkins Credentials**, **Kubernetes Secrets**, or **local environment variables**.

This project follows **DevSecOps best practices**, ensuring:
- No hardcoded passwords, API keys, or tokens are present in source control.
- Jenkins pipelines use credential IDs only (not plaintext secrets).
- Sensitive cluster configurations (like kubeconfig) remain private and stored securely on the control node.
- Public documentation and manifests contain only non-sensitive example values.

---

---

## 👤 Author
**Angad Singh**  
DevOps & Cybersecurity Engineer  
[GitHub: angsgit](https://github.com/angsgit)
