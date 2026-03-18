# ✈️ AI Travel Itinerary Planner

AI Travel Itinerary Planner is an end-to-end LLMOps project that combines generative AI with production-grade engineering. The frontend is built with Streamlit, and the core logic lives in a modular TravelPlanner class that interfaces with the Groq API to invoke a large language model and generate personalized day-trip itineraries based on user-provided city and interests. The app is fully Dockerized using a custom Dockerfile, pushed to a GitHub repository, and deployed on a Google Cloud VM (E2 Standard, 16 GB RAM, Ubuntu 24.04 LTS) running a Minikube Kubernetes cluster. Kubernetes manifests (k8s-deployment.yaml) handle the deployment and service configuration, while sensitive credentials like the Groq API key are injected securely using Kubernetes Secrets — keeping the app stateless and configuration-driven rather than hardcoded.

The project also implements a full ELK Stack observability pipeline deployed as a dedicated logging namespace within the same Kubernetes cluster. Filebeat runs as a DaemonSet to automatically collect logs from all pods and nodes, forwarding them to Logstash for processing and enrichment, which then ships them to Elasticsearch backed by a PersistentVolume for durable log storage. Kibana is exposed via port-forwarding and configured with a filebeat-* index pattern and @timestamp field, enabling real-time log exploration, filtering by kubernetes.container.name, and live cluster monitoring through the Discover and Analytics dashboards — replicating the kind of observability stack used in production AI deployments.

---

## 🏗️ Project Architecture

Layer 1 — User Access
A user opens the app in their browser. Their request hits a Kubernetes Service (streamlit-service) exposed via port-forward on :8501, which routes traffic into the running Streamlit pod inside the cluster.

Layer 2 — Application (default namespace)
The Streamlit app (app.py) receives the city and interests input and passes it to the TravelPlanner class, which makes a live API call to Groq to invoke an LLM (LLaMA/Mixtral) and generate the itinerary. The Groq API key is never hardcoded — it's injected at runtime from a Kubernetes Secret (llmops-secrets), keeping credentials secure and the app stateless.

Layer 3 — Infrastructure (GCP VM)
The entire cluster runs inside a Google Cloud VM (E2 Standard, 16 GB RAM, 256 GB disk, Ubuntu 24.04). Docker is installed on the VM, and eval $(minikube docker-env) points Docker's build context directly at Minikube's internal registry — so when you run docker build, the image is instantly available to Kubernetes without needing to push it to an external registry. The app is deployed via k8s-deployment.yaml which defines the pod spec, resource limits, and service configuration.

Layer 4 — ELK Observability Stack (logging namespace)
All pod logs across the cluster are automatically captured by Filebeat, running as a DaemonSet (meaning one instance per node), which tails /var/log/containers/* in real time. Logs are forwarded to Logstash, which parses, filters, and enriches them before shipping them to Elasticsearch for storage — backed by a PersistentVolume so logs survive pod restarts. Kibana sits on top, exposed on :5601 via port-forward, with a filebeat-* index pattern and @timestamp as the time field. From the Discover panel, developers can filter logs by kubernetes.container.name to isolate activity from any specific pod — the Streamlit app, Logstash, Kibana itself, or any other component.

Layer 5 — Deployment Pipeline
Code lives on GitHub, cloned onto the VM, built into a Docker image, and applied to the cluster in three commands: docker build, kubectl create secret, and kubectl apply -f k8s-deployment.yaml.
