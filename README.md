# ☁️ CloudVault2
### Your Documents. Organized. Accessible. Deployed.

A full-stack document management platform built with **React, Flask, and PostgreSQL**, powered by **Docker, Kubernetes, and Jenkins** for automated deployment.

<p align="center">
  <img src="https://img.shields.io/badge/React-Frontend-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/Flask-Backend-black?style=for-the-badge&logo=flask" />
  <img src="https://img.shields.io/badge/PostgreSQL-Database-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-Containers-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Kubernetes-Orchestration-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" />
  <img src="https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?style=for-the-badge&logo=jenkins&logoColor=white" />
</p>

<p align="center">
  <strong>UPLOAD → ORGANIZE → RETRIEVE → AUTOMATE → DEPLOY</strong>
</p>

---

## 🚀 What is CloudVault2?

CloudVault2 is more than a file upload application. It's a full-stack project that brings together document management and modern deployment practices.

Users can upload, browse, download, and delete documents through a React interface, while the Flask backend handles API operations, PostgreSQL stores document metadata, and Docker and Kubernetes handle deployment.

The best part? **Every push to GitHub can trigger an automated Jenkins pipeline**, taking changes through testing, image publishing, and Kubernetes deployment.

## ✨ Features

| Feature | What it does |
|---|---|
| 📤 Document Upload | Upload documents through a clean web interface |
| 📚 Document Library | View documents, metadata, file sizes, and timestamps |
| ⬇️ Download & Delete | Retrieve or remove stored documents |
| 💚 Health Monitoring | Check backend availability through a health endpoint |
| 🗄️ PostgreSQL | Persist document metadata in a relational database |
| 🐳 Docker | Containerize the Flask backend |
| ☸️ Kubernetes | Deploy and manage application resources |
| ⚙️ Jenkins CI/CD | Automate tests, builds, image publishing, and deployment |
| 🔔 GitHub Webhooks | Trigger CI/CD automatically on repository pushes |

## 🏗️ System Architecture

### Application Architecture

```mermaid
flowchart TD
    U[👤 User] --> FE[React + Vite]
    FE -->|REST API| BE[Flask Backend]
    BE --> DB[(PostgreSQL)]
    BE --> FS[Document File Storage]
    BE --> H[Health Endpoint]
```

### CI/CD Architecture

```mermaid
flowchart TD
    A[GitHub Push] --> B[GitHub Webhook]
    B --> C[ngrok HTTPS Tunnel]
    C --> D[Jenkins Pipeline]
    D --> E[Backend Tests]
    E --> F[Frontend Build]
    F --> G[Docker Image Build]
    G --> I[Docker Hub]
    I --> J[Kubernetes Deployment]
    J --> K[Deployment Verification]
```

**From commit to deployment:** A push to the repository notifies Jenkins through the GitHub webhook. Jenkins checks out the latest code, runs backend tests, builds the frontend, creates and publishes a versioned Docker image, and updates the Kubernetes deployment.

## 🛠️ Built With

| Category | Technologies |
|---|---|
| Frontend | React, Vite, JavaScript |
| Backend | Python, Flask, REST API |
| Database | PostgreSQL |
| Containerization | Docker, Docker Compose |
| Orchestration | Kubernetes, kubectl |
| CI/CD | Jenkins |
| Image Registry | Docker Hub |
| Automation | GitHub Webhooks, ngrok |
| Testing & Build | pytest, npm, ESLint, Vite |

## 📂 Project Structure

```text
CloudVault2/
│
├── backend/
│   ├── app.py
│   ├── init_db.py
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── storage/
│   │   └── documents/
│   └── tests/
│
├── frontend/
│   ├── src/
│   └── package.json
│
├── k8s/
│   ├── postgres.yaml
│   ├── backend.yaml
│   └── backend-service.yaml
│
├── docker-compose.yml
└── Jenkinsfile
```

## ⚡ Getting Started

### Prerequisites

- Python 3.12
- Node.js and npm
- Docker Desktop with Docker Compose
- Git
- PostgreSQL (if running outside Docker)

### 1. Clone the repository

```bash
git clone https://github.com/chowgulesonakshi-lab/CloudVault2.git
cd CloudVault2
```

### 2. Start PostgreSQL

From the project root, open PowerShell:

```powershell
$env:POSTGRES_PASSWORD = "choose-a-local-password"
docker compose up -d db
```

PostgreSQL is exposed on host port `5433`.

### 3. Set up the backend

```powershell
cd backend

py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt

$env:DATABASE_URL = "postgresql://postgres:choose-a-local-password@localhost:5433/cloudvault"

python app.py
```

Backend API: `http://localhost:5000`

### 4. Run the frontend

Open a separate terminal:

```powershell
cd frontend

npm install
npm run dev
```

Frontend: `http://localhost:5173`

## 🔌 API Reference

Base URL: `http://localhost:5000`

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | API welcome message |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/documents/upload` | Upload a document |
| `GET` | `/api/documents` | List documents |
| `GET` | `/api/documents/<id>/download` | Download a document |
| `DELETE` | `/api/documents/<id>` | Delete a document |

### Quick Health Check

```powershell
Invoke-RestMethod http://localhost:5000/api/health
```

### Upload a Document

```powershell
curl.exe -X POST `
  -F "file=@C:\path\to\your\document.pdf" `
  http://localhost:5000/api/documents/upload
```

## 🐳 Containerization with Docker

Build the backend image and start the Compose services:

```powershell
$env:POSTGRES_PASSWORD = "choose-a-local-password"

docker build -t cloudvault-backend:latest backend
docker compose up -d
```

Stop the services:

```powershell
docker compose down
```

PostgreSQL uses the `postgres_data` named volume. Uploaded documents are not currently mounted to a persistent volume in the Compose setup.

## ☸️ Kubernetes Deployment

CloudVault2 runs in its own Kubernetes namespace, keeping its resources isolated from the original CloudVault deployment.

<details>
<summary><strong>View Kubernetes deployment steps</strong></summary>

### 1. Create namespace

```powershell
kubectl create namespace cloudvault2
```

### 2. Configure database credentials

```powershell
$password = Read-Host "Enter PostgreSQL password" -AsSecureString
$plainPassword = [System.Net.NetworkCredential]::new("", $password).Password

kubectl create secret generic cloudvault-db-secret `
  --namespace cloudvault2 `
  --from-literal="POSTGRES_PASSWORD=$plainPassword"

Remove-Variable password, plainPassword
```

### 3. Deploy resources

```powershell
kubectl apply -n cloudvault2 -f k8s/postgres.yaml
kubectl apply -n cloudvault2 -f k8s/backend.yaml
kubectl apply -n cloudvault2 -f k8s/backend-service.yaml
```

### 4. Verify

```powershell
kubectl get pods -n cloudvault2
kubectl get deployments -n cloudvault2
kubectl get services -n cloudvault2
```

</details>

### Deployment Configuration

| Resource | Configuration |
|---|---|
| Namespace | `cloudvault2` |
| Backend | `cloudvault-backend` |
| Database | `cloudvault-db` |
| Backend Service | `cloudvault-backend-service` |
| Service Type | NodePort |
| Backend Port | `5000` |
| NodePort | `30081` |
| Database Port | `5432` |
| Database Service | ClusterIP |
| Database Secret | `cloudvault-db-secret` |

> **Storage note:** PostgreSQL metadata is persisted through a Kubernetes PersistentVolumeClaim. Uploaded document files are stored separately and do not currently use a dedicated persistent Kubernetes volume.

## ⚙️ CI/CD — From Git Push to Kubernetes

CloudVault2 uses a Jenkins pipeline defined in the root-level `Jenkinsfile`.

### The Automated Workflow

| Stage | Action |
|---|---|
| 01 — Checkout | Fetch the latest code from GitHub |
| 02 — Backend Test | Install dependencies and execute pytest |
| 03 — Frontend Build | Install npm packages and generate the production build |
| 04 — Docker Build | Build the backend image |
| 05 — Docker Hub Push | Publish the versioned image |
| 06 — Kubernetes Deploy | Apply manifests and update the deployment |
| 07 — Verify | Check pods, services, rollout status, and deployed image |

### 🔔 Automatic GitHub Trigger

The GitHub webhook integration connects repository activity to Jenkins.

1. A developer pushes changes to the `main` branch.
2. GitHub sends a push event to the configured webhook URL.
3. ngrok provides an HTTPS tunnel to the locally hosted Jenkins server.
4. Jenkins receives the event and triggers `CloudVault2-CI-CD`.
5. The pipeline runs the complete CI/CD workflow.

**Manual fallback:** The pipeline can also be started using **Build Now** in Jenkins.

> **Note:** Because Jenkins runs locally, the computer, Jenkins service, and ngrok tunnel must remain online for automatic webhook triggering. The ngrok public URL may change when a new tunnel session is started; update the GitHub webhook URL if it changes.

### Docker Image Versioning

Each Jenkins build publishes a versioned image:

```text
codecurious06/cloudvault-backend:<BUILD_NUMBER>
```

This provides versioned images for deployment tracking and rollback.

### Jenkins Configuration

The Jenkins pipeline requires:

- Git access to the CloudVault2 repository.
- Python and Node.js environments.
- Docker Desktop and Docker CLI.
- Kubernetes CLI (`kubectl`).
- Docker Hub credentials configured in Jenkins.
- Access to the target Kubernetes cluster.
- Kubernetes configuration for the `docker-desktop` context.

The Docker Hub credentials are managed through Jenkins Credentials rather than being stored in the repository.

### Triggering the Pipeline

The pipeline can be started manually using **Build Now** in Jenkins.

GitHub webhook-based automatic triggering is optional and is