# 🚀 E-Flow Microservices — CI/CD Pipeline Setup

> **Project:** E-Flow (Microservices Architecture)
> **Stack:** React Frontend + Node.js Backend Microservices
> **Registry:** Harbor (Private)
> **Cloud:** AWS (EKS + S3 + CloudFront)
> **CI/CD:** Jenkins
> **Monitoring:** Prometheus + Grafana
> **Code Quality:** SonarQube

---

## 📁 Folder Structure

```
eflow-cicd/
├── frontend/
│   ├── Jenkinsfile                  ← React pipeline (7 stages)
│   ├── Dockerfile                   ← Multi-stage React build
│   ├── nginx.conf                   ← Nginx config for React Router
│   └── sonar-project.properties     ← SonarQube config
│
├── backend/
│   ├── Jenkinsfile                  ← Node.js microservice pipeline (12 stages)
│   ├── Dockerfile                   ← Multi-stage Node.js build
│   └── sonar-project.properties     ← SonarQube config
│
├── docs/
│   └── jenkins-credentials-setup.txt ← Jenkins credentials guide
│
└── README.md                        ← You are here 📍
```

---

## 🎨 Frontend Pipeline — 7 Stages

```
GitHub Checkout
      ↓
Install Dependencies  (npm install)
      ↓
Build Frontend        (npm run build → dist/)
      ↓
SonarQube Scan        (code quality check)
      ↓
SonarQube Quality Gate (pass/fail decision)
      ↓
Deploy to S3          (aws s3 sync)
      ↓
CloudFront Invalidation (cache clear)
```

### Environment Variables to Update (frontend/Jenkinsfile)

| Variable | Description | Example |
|---|---|---|
| `S3_BUCKET` | Your S3 bucket name | `eflow-frontend-bucket` |
| `CF_DIST_ID` | CloudFront Distribution ID | `E1ABCDEF123456` |
| `AWS_REGION` | AWS region | `eu-north-1` |
| `BUILD_DIR` | Build output folder | `dist` (Vite) or `build` (CRA) |

---

## ⚙️ Backend Pipeline — 12 Stages

```
GitHub Checkout
      ↓
Install Dependencies  (npm install)
      ↓
SonarQube Scan
      ↓
SonarQube Quality Gate
      ↓
Docker Image Build
      ↓
Harbor Login
      ↓
Tag Docker Image
      ↓
Push Image to Harbor
      ↓
Deploy to AWS EKS     (kubectl rolling update)
      ↓
Check Service External IP
      ↓
Prometheus Monitoring Check
      ↓
Grafana Dashboard Verification ✅
```

### Environment Variables to Update (backend/Jenkinsfile)

| Variable | Description | Example |
|---|---|---|
| `SERVICE_NAME` | Microservice name | `userservice` |
| `REPO_URL` | GitHub repo URL | `https://github.com/...` |
| `HARBOR_REGISTRY` | Harbor server hostname | `harbor-node1.com` |
| `HARBOR_PROJECT` | Harbor project name | `eflow` |
| `EKS_CLUSTER_NAME` | EKS cluster name | `eflow-cluster` |
| `AWS_REGION` | AWS region | `eu-north-1` |
| `K8S_NAMESPACE` | Kubernetes namespace | `eflow` |
| `PROMETHEUS_URL` | Prometheus server URL | `http://1.2.3.4:9090` |
| `GRAFANA_URL` | Grafana server URL | `http://1.2.3.4:3000` |

---

## 🔑 Jenkins Credentials Required

Go to: **Jenkins → Manage Jenkins → Credentials → System → Global → Add Credential**

| Credential ID | Type | Used In |
|---|---|---|
| `github-creds` | Username + Password (PAT) | Both pipelines — Git checkout |
| `aws-creds` | AWS Credentials | Frontend (S3/CF) + Backend (EKS) |
| `harbor-creds` | Username + Password | Backend — Harbor login/push |
| `grafana-creds` | Username + Password | Backend — Grafana health check |
| `SonarQube` | Secret Text (Token) | Both — SonarQube plugin |

> 📄 Full setup steps in: `docs/jenkins-credentials-setup.txt`

---

## 🛠️ Jenkins Agent Prerequisites

Run these on your Jenkins server/agent:

```bash
# 1. Node.js (via NVM or Jenkins NodeJS plugin)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y

# 2. Docker
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

# 3. AWS CLI
sudo apt install awscli -y

# 4. kubectl
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# 5. sonar-scanner (for npx usage - auto handled)
# OR install globally:
npm install -g sonar-scanner
```

---

## 🐳 Harbor Registry Setup

```bash
# Docker daemon - add Harbor as insecure registry
sudo nano /etc/docker/daemon.json

# Add:
{
  "insecure-registries": ["harbor-node1.com"]
}

sudo systemctl restart docker

# Copy Harbor SSL cert to Docker certs.d
sudo mkdir -p /etc/docker/certs.d/harbor-node1.com/
sudo cp ca.crt /etc/docker/certs.d/harbor-node1.com/
```

---

## 🔁 Adding a New Microservice

Just copy `backend/Jenkinsfile` and change **2 lines only**:

```groovy
SERVICE_NAME = 'authservice'      // ← new service name
REPO_URL     = 'https://github.com/peakyblinder0509/eflow-authservice'
```

Supported services:
- `userservice`
- `authservice`
- `courseservice`
- `contentservice`
- `instituteservice`

---

## 📊 Monitoring Stack

| Tool | URL | Default Port |
|---|---|---|
| Prometheus | `http://<EC2-IP>:9090` | 9090 |
| Grafana | `http://<EC2-IP>:3000` | 3000 |
| Node Exporter | `http://<EC2-IP>:9100` | 9100 |
| cAdvisor | `http://<EC2-IP>:8080` | 8080 |

**Grafana Dashboard Import IDs:**
- Node Exporter: `1860`
- Docker / cAdvisor: `193`

---

## ⚡ Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/peakyblinder0509/eflow-cicd

# 2. Create Jenkins Pipeline Job
#    → New Item → Pipeline → Pipeline script from SCM
#    → SCM: Git → Repo URL → Branch: main
#    → Script Path: frontend/Jenkinsfile  (or backend/Jenkinsfile)

# 3. Update environment variables in Jenkinsfile

# 4. Add all credentials in Jenkins

# 5. Trigger Build → Watch the magic! 🎉
```

---

## 🐞 Common Issues & Fixes

| Issue | Fix |
|---|---|
| `x509: certificate` Harbor error | Copy `ca.crt` to `/etc/docker/certs.d/harbor-node1.com/` |
| `ImagePullBackOff` in EKS | Check Harbor robot account permissions, imagePullSecret in namespace |
| SonarQube Quality Gate timeout | Increase timeout to 5 min, check webhook config in SonarQube |
| `npm: not found` in Jenkins | Install NodeJS plugin → configure NodeJS-18 tool |
| `aws: command not found` | Install AWS CLI on Jenkins agent |
| `kubectl: command not found` | Install kubectl + run `aws eks update-kubeconfig` |

---

## 👤 Author

**Ikram S**
AWS DevOps / Cloud Engineer
Trupp Global Technologies (TGTI105)
GitHub: [@peakyblinder0509](https://github.com/peakyblinder0509)

---

> 💡 **Tip:** Use Jenkins Blue Ocean plugin for a visual pipeline view!
