
# Kubernetes DevOps Project with Jenkins, Docker, Helm, and Monitoring

This project demonstrates an end-to-end CI/CD pipeline deploying a microservices-based application on AWS EKS using Jenkins, Docker, Helm, Prometheus, and Grafana with Dev, Stage, and Prod environments.

---

## 🚀 Manual Setup Instructions

### 1. Prerequisites

Install the following tools on your local or cloud VM:
- AWS CLI
- kubectl
- Docker
- Helm
- Jenkins
- Git

---

### 2. Setup AWS and EKS Cluster

```bash
aws configure
eksctl create cluster --name devops-cluster --region us-east-1 --nodes 2
aws eks update-kubeconfig --name devops-cluster --region us-east-1
```

---

### 3. Create Kubernetes Namespaces

```bash
kubectl create ns dev
kubectl create ns stage
kubectl create ns prod
kubectl create ns monitoring
```

---

### 4. Build and Push Docker Images

```bash
cd services/product-service
docker build -t product-service .

# Push to ECR
aws ecr create-repository --repository-name product-service
aws ecr get-login-password | docker login --username AWS --password-stdin <your-ecr-url>
docker tag product-service:latest <your-ecr-url>:latest
docker push <your-ecr-url>:latest
```

---

### 5. Deploy Using Helm

```bash
# Dev
helm upgrade --install product-service ./helm-charts/product-service -f ./helm-charts/product-service/values-dev.yaml -n dev

# Stage
helm upgrade --install product-service ./helm-charts/product-service -f ./helm-charts/product-service/values-stage.yaml -n stage

# Prod
helm upgrade --install product-service ./helm-charts/product-service -f ./helm-charts/product-service/values-prod.yaml -n prod
```

---

### 6. Setup Jenkins on EC2 (Optional)

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update && sudo apt install jenkins -y
```

Then open Jenkins UI and set up the pipeline using the `Jenkinsfile`.

---

### 7. Monitoring Setup

```bash
# Prometheus & Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring -f monitoring/prometheus-values.yaml
helm install grafana grafana/grafana -n monitoring -f monitoring/grafana-values.yaml
```

---

### 8. Access Grafana Dashboard

```bash
kubectl port-forward svc/grafana 3000:80 -n monitoring
```

Login: `admin / prom-operator`

Import the dashboards from `monitoring/dashboards/`.

---

### 9. Manage Secrets & Configs

```bash
kubectl create secret generic db-secret --from-literal=DB_PASSWORD=admin123 -n dev
kubectl create configmap app-config --from-literal=ENV=dev -n dev
```

---

## ✅ Summary

This project covers:
- CI/CD with Jenkins
- Containerization with Docker
- Multi-environment deployment with Helm
- Monitoring with Prometheus and Grafana
- Secrets and config management
