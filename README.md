# aks-microservice-
Python Flask microservice deployed to Azure Kubernetes Service (AKS)

# 🚀 AKS Microservice Lab

Deploy a Python Flask microservice to Azure Kubernetes Service (AKS) using Docker and Kubernetes. This lab is designed for cloud architects, developers, and community builders who want to learn container orchestration on Azure.

---

## 🧩 Overview

You’ll build a simple Python Flask app, containerize it with Docker, push it to Docker Hub, and deploy it to AKS using Kubernetes manifests. By the end, you’ll have a fully functional microservice running in the cloud with monitoring and scaling capabilities.

---

## 🛠️ Prerequisites

Ensure the following tools are installed and configured:

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- Docker Hub account
- Azure account

---

## 📁 Step 1: Create the Flask App

### 1.1 Create a project folder
```powershell
mkdir aks-microservice
cd aks-microservice
```

### 1.2 Create `app.py`
```powershell
@"
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from AKS!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
"@ | Out-File -Encoding utf8 app.py
```

### 1.3 Create `requirements.txt`
```powershell
"flask" | Out-File -Encoding utf8 requirements.txt
```

### 1.4 Create `Dockerfile`
```powershell
@"
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 80
CMD ["python", "app.py"]
"@ | Out-File -Encoding utf8 Dockerfile
```

---

## 🐳 Step 2: Build and Push Docker Image

### 2.1 Build the image
```powershell
docker build --load -t aks-microservice .
```

### 2.2 Tag the image
```powershell
docker tag aks-microservice josephjem2/aks-microservice:v1
```

### 2.3 Log in to Docker Hub
```powershell
docker login
```

### 2.4 Push the image
```powershell
docker push josephjem2/aks-microservice:v1
```

---

## ☁️ Step 3: Set Up Azure AKS

### 3.1 Log in to Azure
```powershell
az login
```

If needed:
```powershell
az login --tenant <your-tenant-id>
```

### 3.2 Register AKS resource provider
```powershell
az provider register --namespace Microsoft.ContainerService
```

### 3.3 Create a resource group
```powershell
az group create --name aks-lab-rg --location eastus
```

### 3.4 Create the AKS cluster
```powershell
az aks create --resource-group aks-lab-rg --name aks-lab-cluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
```

### 3.5 Connect `kubectl` to AKS
```powershell
az aks get-credentials --resource-group aks-lab-rg --name aks-lab-cluster
```

---

## 📦 Step 4: Deploy to AKS

### 4.1 Create `deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-microservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: aks-microservice
  template:
    metadata:
      labels:
        app: aks-microservice
    spec:
      containers:
      - name: aks-microservice
        image: josephjem2/aks-microservice:v1
        ports:
        - containerPort: 80
```

### 4.2 Create `service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: aks-microservice-service
spec:
  type: LoadBalancer
  selector:
    app: aks-microservice
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Save both files in your project folder.

---

### 4.3 Apply the deployment
```powershell
kubectl apply -f deployment.yaml
```

### 4.4 Apply the service
```powershell
kubectl apply -f service.yaml
```

---

## 🌐 Step 5: Access the App

### 5.1 Get the external IP
```powershell
kubectl get service aks-microservice-service
```

Wait until the `EXTERNAL-IP` field is populated, then open it in your browser.

---

## 📊 Step 6: Monitor the App

### 6.1 View pods
```powershell
kubectl get pods
```

### 6.2 View logs
```powershell
kubectl logs <pod-name>
```

### 6.3 View metrics (if monitoring enabled)
```powershell
kubectl top pods
```

---

## 🧹 Step 7: Clean Up (Optional)
```powershell
az group delete --name aks-lab-rg --yes --no-wait
```

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙌 Author

Created by Joseph Essomba — Cloud Solutions Architect, strategic facilitator, and community builder.  


