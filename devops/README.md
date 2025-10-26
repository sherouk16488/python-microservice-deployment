# DevOps Project Deployment Guide

This guide documents the complete process of deploying a Python microservice using **Docker**, **Terraform**, **GKE (Google Kubernetes Engine)**, and **Cloud Build CI/CD**.

---

##  Task 1 – Clone Git Repository

# 1. Create a folder for your projects
mkdir -p ~/projects
cd ~/projects

# 2. Clone the Microservices repository
git clone https://github.com/sameh-Tawfiq/Microservices.git

# 3. Navigate to the repo
cd Microservices
ls -la

# 4. Verify the remote link
git remote -v
```

✅ Folder created:  
`C:\Users\s.mostafa\projects\Microservices`



---

##  Task 2 – Dockerize the Application

### Steps

1. Open Docker Cloud.  
2. Clone the repository again (if needed):


git clone https://github.com/sameh-Tawfiq/Microservices.git
cd Microservices
ls
```

### Create `Dockerfile`

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY . /app

RUN pip install --upgrade pip && pip install --no-cache-dir -r requirements.txt

EXPOSE 8080

CMD ["python", "run.py"]
```

### Build & Run

```bash
docker build -t microservice-app .
sudo docker run -d -p 8080:8080 microservice-app

Accessible URL
http://localhost:8080

Hint
Use -p 80:8080 in production  if you want users to access via port 80 (HTTP default).


---

##  Task 3 – Terraform Setup for GKE

### Initialize Terraform Environment

```bash
mkdir terraform-gke-quickstart
cd terraform-gke-quickstart

gcloud auth application-default login
gcloud config set project gkepwc
```

### Define Terraform Files

- `provider.tf` → Define provider and backend  
- `variables.tf` → Input variables  
- `terraform.tfvars` → Variable values  
- `main.tf` → GKE cluster and node pool configuration  

### Terraform Commands


terraform init
terraform plan
terraform apply
```


---

##  Task 4 – Deploy the Microservice


# Rebuild image with updated requirements.txt
docker build -t us-west1-docker.pkg.dev/gkepwc/my-app-repo/my-app:v1.1 .

# Push to Artifact Registry
docker push us-west1-docker.pkg.dev/gkepwc/my-app-repo/my-app:v1.1

# Apply Kubernetes deployment
kubectl apply -f deployment.yaml

# Verify pod status
kubectl get pods
```



---

## Task 5 – Expose the Service Publicly

### Create `service.yaml`

A **LoadBalancer** service exposing the application externally.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-microservice-service
spec:
  type: LoadBalancer
  selector:
    app: python-microservice
  ports:
    - port: 80
      targetPort: 8080
```

### Apply and Verify


kubectl apply -f service.yaml
kubectl get service python-microservice-service --watch
```

✅ **External IP:** `34.182.53.25`  
✅ **Test Connection:**


curl http://34.182.53.25
```

**Result:** `Hello from GKE! (v1.0)`



---

## Task 6 – CI/CD Pipeline with Cloud Build

### Create Cloud Build Configuration (`cloudbuild.yaml`)

This pipeline automates build, push, and deploy steps:

1. **Build** Docker image  
2. **Push** to Artifact Registry (`my-app-repo`)  
3. **Deploy** to Cloud Run (`python-web-app`, region `us-west1`)



### Steps

- Copy `Dockerfile` and `cloudbuild.yaml` into your local repo.  
- Merge changes using GitHub Desktop.  
- Fork repository to your GitHub account.  
- In GCP → Cloud Build → **Create Trigger**, link to your repo path.






Task 7 – Monitoring Stack Implementation

1 – Clone the kube-prometheus Repository
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
2 – Apply the Setup Manifests
kubectl apply --server-side -f manifests/setup/
	Wait a few seconds for the CRDs to be created, then confirm
kubectl get crds | grep Prometheus
3 – Deploy the Monitoring Stack
	After the CRDs are installed, apply the main manifests to deploy all components.
kubectl apply -f manifests/
this deploys:
•	Prometheus Operator
•	Prometheus
•	Alertmanager
•	Grafana
•	Kube State Metrics
•	Node Exporter
4 - Verify deployment
kubectl get pods -n monitoring

5 -Access Grafana Dashboard
kubectl --namespace monitoring port-forward svc/grafana 3000
