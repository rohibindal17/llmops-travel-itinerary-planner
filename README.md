# ✈️ AI Travel Itinerary Planner

> An LLM-powered travel planning app built with **Streamlit** and **Groq**, deployed on **Kubernetes** (Minikube on GCP), and monitored end-to-end with the **ELK Stack** (Elasticsearch, Logstash, Kibana, Filebeat).

---

## 📌 Recommended Repo Name

**`llmops-travel-itinerary-planner`**

> Reflects both the LLMOps deployment pipeline (Docker → Kubernetes → ELK monitoring) and the core product (AI travel itinerary generation).

---

## 🧠 What It Does

Users enter a **city** and their **interests**, and the app generates a personalized day-trip itinerary using a large language model via the Groq API. The entire application is containerized, orchestrated with Kubernetes, and observable through a full ELK logging stack.

---

## 🏗️ Project Architecture

```
User (Browser)
    │
    ▼
Streamlit App (app.py)
    │  uses
    ▼
TravelPlanner (src/core/planner.py)  ──►  Groq LLM API
    │
    ▼
Kubernetes Cluster (Minikube on GCP VM)
    │
    ├── streamlit-service (NodePort/LoadBalancer)
    │
    └── logging namespace
            ├── Filebeat      (collects pod logs)
            ├── Logstash      (processes & forwards logs)
            ├── Elasticsearch (stores logs)
            └── Kibana        (visualizes logs)
```

---

## 🗂️ Repository Structure

```
├── src/
│   └── core/
│       └── planner.py          # TravelPlanner class (LLM logic)
├── app.py                      # Streamlit frontend
├── Dockerfile                  # Containerizes the Streamlit app
├── k8s-deployment.yaml         # Kubernetes Deployment + Service for the app
├── elasticsearch.yaml          # Elasticsearch K8s manifest
├── kibana.yaml                 # Kibana K8s manifest
├── logstash.yaml               # Logstash K8s manifest
├── filebeat.yaml               # Filebeat DaemonSet manifest
├── requirements.txt            # Python dependencies
├── setup.py                    # Package setup
└── .gitignore
```

---

## 🚀 Getting Started

### Prerequisites

- Google Cloud VM (E2 Standard, 16 GB RAM, 256 GB disk, Ubuntu 24.04 LTS)
- Docker installed on the VM
- Minikube + kubectl installed
- A [Groq API Key](https://console.groq.com/)
- Your code pushed to GitHub

---

### 1. Clone the Repository

```bash
git clone https://github.com/data-guru0/AI-TRAVEL-ITINEARY-PLANNER.git
cd AI-TRAVEL-ITINEARY-PLANNER
```

---

### 2. Configure VM — Install Docker

Install Docker following the [official Docker docs](https://docs.docker.com/engine/install/ubuntu/), then run post-installation steps so Docker works without `sudo`. Enable Docker on boot:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

Verify:

```bash
systemctl status docker   # Should show "active (running)"
docker ps
```

---

### 3. Set Up Minikube

Install Minikube (Linux x86 binary) from [minikube.sigs.k8s.io](https://minikube.sigs.k8s.io/docs/start/), then:

```bash
minikube start
```

Install `kubectl` via Snap:

```bash
sudo snap install kubectl --classic
kubectl version --client
```

Verify the cluster:

```bash
minikube status
kubectl get nodes
kubectl cluster-info
```

---

### 4. Build & Deploy the App

```bash
# Point Docker to Minikube's internal registry
eval $(minikube docker-env)

# Build the Docker image
docker build -t streamlit-app:latest .

# Create Kubernetes secret with your Groq API key
kubectl create secret generic llmops-secrets \
  --from-literal=GROQ_API_KEY="<your-groq-api-key>"

# Deploy the app
kubectl apply -f k8s-deployment.yaml

# Check pod status
kubectl get pods
```

Once pods are running, forward the port to access the app:

```bash
kubectl port-forward svc/streamlit-service 8501:80 --address 0.0.0.0
```

Open your browser at `http://<your-vm-external-ip>:8501` 🎉

---

## 📊 ELK Stack Monitoring Setup

All application and cluster logs are collected and visualized using the ELK stack deployed inside the same Kubernetes cluster.

### Step 1 — Create the Logging Namespace

```bash
kubectl create namespace logging
```

### Step 2 — Deploy Elasticsearch

```bash
kubectl apply -f elasticsearch.yaml
kubectl get pods -n logging
kubectl get pvc -n logging   # Should be in "Bound" state
```

### Step 3 — Deploy Kibana

```bash
kubectl apply -f kibana.yaml
kubectl get pods -n logging

# Forward Kibana to your browser
kubectl port-forward -n logging svc/kibana 5601:5601 --address 0.0.0.0
```

Access Kibana at `http://<your-vm-external-ip>:5601`

### Step 4 — Deploy Logstash

```bash
kubectl apply -f logstash.yaml
kubectl get pods -n logging
```

### Step 5 — Deploy Filebeat

```bash
kubectl apply -f filebeat.yaml
kubectl get all -n logging
```

### Step 6 — Configure Index Pattern in Kibana

1. Open Kibana → click **"Explore on my own"**
2. Go to **Stack Management → Index Patterns**
3. Create a new pattern:
   - **Pattern name:** `filebeat-*`
   - **Timestamp field:** `@timestamp`
4. Click **Create Index Pattern**

### Step 7 — Explore Logs

Navigate to **Analytics → Discover** in Kibana. Filter by `kubernetes.container.name` to isolate logs from specific pods (e.g., your Streamlit app, Logstash, Kibana).

---

## 🔧 Environment Variables

| Variable | Description |
|---|---|
| `GROQ_API_KEY` | Your Groq API key for LLM inference |

Managed as a Kubernetes Secret — never hardcoded.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | Streamlit |
| **LLM Backend** | Groq API |
| **Containerization** | Docker |
| **Orchestration** | Kubernetes (Minikube) |
| **Infrastructure** | Google Cloud VM (Ubuntu 24.04) |
| **Log Collection** | Filebeat |
| **Log Processing** | Logstash |
| **Log Storage** | Elasticsearch |
| **Log Visualization** | Kibana |

---

## 📎 Git Configuration (for VM)

```bash
git config --global user.email "your-email@example.com"
git config --global user.name "your-username"
```

When pushing, use your GitHub **Personal Access Token** as the password (it won't be visible when typed).
