
# 🚀 End-to-End CI/CD Deployment of Flask App on AWS EKS

This project demonstrates a full CI/CD pipeline to build, test, containerize, and deploy a simple Flask app to AWS EKS using Jenkins, Docker, and Terraform.

---

## 🛠️ Tech Stack

| Category          | Tools Used                                   |
|------------------|----------------------------------------------|
| CI/CD            | Jenkins, Docker Hub                          |
| Containerization | Docker, Docker Compose                       |
| Cloud            | AWS (EKS, EC2, VPC, Subnets, IAM)            |
| Infrastructure   | Terraform                                     |
| Kubernetes       | EKS, kubectl, LoadBalancer Service           |
| App Framework    | Python Flask                                 |

---

## 🧱 Architecture Diagram

```

\[ GitHub ]
|
v
\[ Jenkins ] ---> \[ Docker Build ] ---> \[ Docker Push to DockerHub ]
|
v
\[ Terraform Infra Setup ] ---> \[ EKS Cluster + Nodes ]
|
v
\[ kubectl Apply ] ---> \[ Flask App Deployed on EKS ]

````

---

## 🐳 Docker Overview

- A `Dockerfile` defines the app runtime
- Tagged and pushed as `vaibhavi210/flask-app:<build-number>`
- Docker Compose used for local multi-container testing

---

## ⚙️ CI/CD Workflow

1. **Source Code Checkout** from GitHub
2. **Docker Build & Push** to DockerHub using Jenkins
3. **Infrastructure Provisioning** using Terraform
4. **Kubernetes Deployment** with `kubectl`
5. **LoadBalancer Service** exposes app on the internet

---

## 📦 Infrastructure Provisioning (Terraform)

```bash
terraform init
terraform apply -auto-approve
````

Creates:

* VPC, Subnets
* Internet Gateway
* EKS Cluster & Node Groups

---

## ☸️ Deployment to EKS

```bash
aws eks update-kubeconfig --name devops-cluster --region ap-south-1
kubectl apply -f deployment.yaml
kubectl get svc flask-app
```

Access the app using the `EXTERNAL-IP:5000`

---

## 🧹 Cleanup

```bash
terraform destroy -auto-approve
```

Deletes all AWS resources to avoid billing.

---

## ✅ Output

```json
{
  "message": "Hello from Flask on EKS!"
}
```

---

## 📸 Screenshots

> You can add:

* `kubectl get pods` output
![alt text](<Screenshot 2025-07-14 200039.png>)

* Jenkins pipeline stages
![alt text](<Screenshot 2025-07-14 195905.png>)

---

## 🤝 Author

**Vaibhavi Khatri**
📧 \[[Vaibhavikhatri21004@gmail.com](mailto:vaibhavikhatri21004@gmail.com)]


---

## 📌 License

MIT License

```

---


