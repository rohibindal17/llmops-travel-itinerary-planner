# ✈️ AI Travel Itinerary Planner

> An LLM-powered travel planning app built with **Streamlit** and **Groq**, deployed on **Kubernetes** (Minikube on GCP), and monitored end-to-end with the **ELK Stack** (Elasticsearch, Logstash, Kibana, Filebeat).

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
