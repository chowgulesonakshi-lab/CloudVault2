# ☁️ CloudVault2 – Document Management Platform

A full-stack document management platform built to simplify file storage, organization, and retrieval through a modern web interface and a RESTful backend.

CloudVault2 combines **React, Flask, PostgreSQL, Docker, Kubernetes, and Jenkins** to demonstrate full-stack development, containerization, persistent metadata storage, deployment orchestration, and automated CI/CD.

<p align="center">
  <strong>Upload · Organize · Retrieve · Deploy</strong>
</p>

---

## ✨ Features

- **📤 Document Upload** — Upload documents through an intuitive React interface.
- **📚 Document Library** — Browse stored documents with file size, content type, status, and upload timestamp.
- **⬇️ Download & Delete** — Retrieve stored files or remove documents from the library.
- **💚 Health Monitoring** — Check API availability through a dedicated health endpoint.
- **🗄️ PostgreSQL Integration** — Store document metadata in a relational database.
- **🐳 Containerized Backend** — Package and run the Flask API using Docker.
- **☸️ Kubernetes Deployment** — Deploy backend and database resources using Kubernetes manifests.
- **⚙️ Automated CI/CD** — Jenkins automates testing, frontend builds, Docker image creation, publishing, and Kubernetes deployment.

## 🏗️ Architecture

```mermaid
flowchart TD
    A[User] --> B[React + Vite Frontend]
    B -->|HTTP / REST API| C[Flask Backend]
    C --> D[(PostgreSQL)]
    C --> E[Document File Storage]

    C --> F[Health Endpoint]

    G[GitHub Repository] --> H[Jenkins Pipeline]
    H --> I[Backend Tests]
    I --> J[Frontend Build]
    J --> K[Docker Image Build]
    K --> L[Docker Hub]
    L --> M[Kubernetes Deployment]
    M --> C
    M --> D
```

**How it works:** The frontend communicates with the Flask REST API. The backend handles document operations, stores metadata in PostgreSQL, and saves uploaded files in backend storage. Docker packages the backend, while Kubernetes manages the application and database deployments. Jenkins automates the CI/CD workflow from source checkout to deployment verification.

## 🛠️ Tech Stack

| Layer | Technologies |
|---|---|
| Frontend | React, Vite, JavaScript |
| Backend | Python, Flask, REST API |
| Database | PostgreSQL |
| Containerization | Docker, Docker Compose |
| Orchestration | Kubernetes, kubectl |
| CI/CD | Jenkins |
| Image Registry | Docker Hub |
| Testing & Build | pytest, npm, ESLint, Vite |

## 📁 Project Structure

```text
CloudVault2/
├── backend/
│   ├── app.py
│   ├── init_db.py
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── storage/
│   │   └── documents/
│   └── tests/
├── frontend/
│   ├── src/
│   └── package.json
├── k8s/
│   ├── postgres.yaml
│   ├── backend.yaml
│   └── backend-service.yaml
├── docker-compose.yml
└── Jenkinsfile
```

## 🚀 Getting Started

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

The Compose database is named `cloudvault` and is exposed on port `5433` on the host.

### 3. Start the Flask backend

Open a separate PowerShell terminal:

```powershell
cd backend
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt

$env:DATABASE_URL = "postgresql://postgres:choose-a-local-password@localhost:5433/cloudvault"

python app.py
```

The API will be available at:

`http://localhost:5000`

Use the same local database password in both terminals.

### 4. Start the React frontend

Open another terminal from the project root:

```powershell
cd frontend
npm install
npm run dev
```

Open the local URL printed by Vite, typically:

`http://localhost:5173`

Keep the backend running while using the frontend.

## 🔌 API Reference

Base URL: `http://localhost:5000`

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | API welcome message |
| `GET` | `/api/health` | Check API health |
| `POST` | `/api/documents/upload` | Upload a document |
| `GET` | `/api/documents` | List stored documents |
| `GET` | `/api/documents/<id>/download` | Download a document |
| `DELETE` | `/api/documents/<id>` | Delete a document |

### Example: Check API Health

```powershell
Invoke-RestMethod http://localhost:5000/api/health
```

### Example: Upload a Document

```powershell
curl.exe -X POST `
  -F "file=@C:\path\to\your\document.pdf" `
  http://localhost:5000/api/documents/upload
```

The upload endpoint accepts a multipart form field named `file` and returns `201 Created` on success.

## 🐳 Docker Deployment

Build the backend image and start the Compose services:

```powershell
$env:POSTGRES_PASSWORD = "choose-a-local-password"

docker build -t cloudvault-backend:latest backend
docker compose up -d
```

This starts the backend and PostgreSQL. Run the frontend separately using the development steps above.

Stop the services:

```powershell
docker compose down
```

PostgreSQL uses the `postgres_data` named volume. Uploaded documents are not currently mounted to a persistent volume in the Compose setup, so container replacement may result in file loss.

## ☸️ Kubernetes Deployment

CloudVault2 uses Kubernetes manifests to deploy the Flask backend and PostgreSQL database.

The deployment runs in a dedicated namespace:

`cloudvault2`

This keeps CloudVault2's Kubernetes resources separate from the original CloudVault deployment.

### 1. Create the namespace

```powershell
kubectl create namespace cloudvault2
```

### 2. Configure database credentials

Create a Kubernetes Secret for the PostgreSQL password:

```powershell
$password = Read-Host "Enter PostgreSQL password" -AsSecureString
$plainPassword = [System.Net.NetworkCredential]::new("", $password).Password

kubectl create secret generic cloudvault-db-secret `
  --namespace cloudvault2 `
  --from-literal="POSTGRES_PASSWORD=$plainPassword"

Remove-Variable password, plainPassword
```

### 3. Deploy the database and backend

```powershell
kubectl apply -n cloudvault2 -f k8s/postgres.yaml
kubectl apply -n cloudvault2 -f k8s/backend.yaml
kubectl apply -n cloudvault2 -f k8s/backend-service.yaml
```

### 4. Verify the deployment

```powershell
kubectl get pods -n cloudvault2
kubectl get deployments -n cloudvault2
kubectl get services -n cloudvault2
```

### Kubernetes Configuration

| Resource | Configuration |
|---|---|
| Namespace | `cloudvault2` |
| Backend Deployment | `cloudvault-backend` |
| Database Deployment | `cloudvault-db` |
| Backend Service | `cloudvault-backend-service` |
| Backend Service Type | NodePort |
| Backend Container Port | `5000` |
| Backend NodePort | `30081` |
| Database Service | ClusterIP |
| Database Port | `5432` |
| Database Secret | `cloudvault-db-secret` |

The backend uses PostgreSQL environment variables sourced from the Kubernetes Secret and connects to the database through the `cloudvault-db` Kubernetes service.

The NodePort is configured as `30081`. Actual external accessibility depends on the Kubernetes environment and networking configuration.

> **Storage note:** PostgreSQL uses persistent volume storage through a Kubernetes PersistentVolumeClaim. Uploaded document files are stored separately by the backend and are not currently backed by a dedicated persistent Kubernetes volume. Consider persistent file storage before using the application for important documents.

## ⚙️ CI/CD Pipeline with Jenkins

CloudVault2 uses **Jenkins** to automate the application build, testing, containerization, and Kubernetes deployment process.

The pipeline is defined in the root-level `Jenkinsfile` and retrieves source code directly from the CloudVault2 GitHub repository.

### Pipeline Workflow

```mermaid
flowchart TD
    A[GitHub Repository] --> B[Checkout]
    B --> C[Backend Tests]
    C --> D[Frontend Build]
    D --> E[Docker Build]
    E --> F[Docker Hub Push]
    F --> G[Kubernetes Deploy]
    G --> H[Deployment Verification]
```

### Pipeline Stages

| Stage | Description |
|---|---|
| Checkout | Clone the latest source code from the `main` branch. |
| Backend Test | Install Python dependencies and execute pytest. |
| Frontend Build | Install npm dependencies and generate the production build. |
| Docker Build | Build the backend Docker image using a Jenkins build-number tag. |
| Docker Hub Push | Authenticate and publish the image to Docker Hub. |
| Kubernetes Deploy | Apply Kubernetes manifests and update the backend deployment. |
| Verify | Check Kubernetes pods, services, deployment status, and deployed image. |

### Docker Image Versioning

Docker images are tagged using Jenkins build numbers:

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

GitHub webhook-based automatic triggering is optional and is not required for manual pipeline execution.

### Build Retention

Jenkins build history is configured to retain:

- A maximum of 10 builds.
- A maximum history period of 14 days.

This helps control the growth of Jenkins build history.

## 🧪 Testing & Quality Checks

### Backend Tests

Run from the `backend/` directory with the virtual environment activated and PostgreSQL available:

```powershell
python -m pytest
```

### Frontend Checks

Run from the `frontend/` directory:

```powershell
npm run lint
npm run build
```

The Jenkins pipeline automatically executes the backend tests and frontend production build before creating and publishing the Docker image.

## 🔐 Security & Reliability

CloudVault2 is a development project. Several production-hardening steps remain:

- Add authentication and authorization.
- Validate uploads and enforce file-size and file-type restrictions.
- Use secure configuration and secret management.
- Add persistent storage for uploaded documents.
- Configure production networking and HTTPS.
- Add monitoring, logging, backups, and appropriate error handling.
- Implement controlled access to the Kubernetes cluster and Jenkins instance.

Do not expose the current unauthenticated API to untrusted networks or use it to store sensitive documents without implementing suitable security controls.

## 🎯 Project Goals

CloudVault2 brings together full-stack development and infrastructure automation in one project.

It demonstrates hands-on experience with:

- Building and integrating a React frontend with a Flask REST API.
- Managing relational data with PostgreSQL.
- Packaging services with Docker and Docker Compose.
- Deploying containerized applications using Kubernetes.
- Automating backend testing and frontend production builds.
- Publishing versioned Docker images to Docker Hub.
- Implementing an end-to-end Jenkins CI/CD pipeline.
- Managing isolated Kubernetes deployments and deployment verification.

## 👩‍💻 Author

**Sonakshi Chowgule**

Computer Science & Engineering  
MIT World Peace University, Pune

---

<p align="center">
  <em>CloudVault2 — Making document management simple, structured, and deployable.</em>
</p>
