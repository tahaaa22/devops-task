# DevOps Internship Assessment – SilverKey

## Project Overview

This repository contains the deployment setup for a simple ToDo application using modern DevOps practices. It includes:

- Dockerized microservices
- GitHub Actions for CI/CD
- Ansible for server provisioning
- Docker Compose for orchestration
- Watchtower for auto-updates
- Bonus: Kubernetes with ArgoCD for GitOps-based deployment

---

##  Features

- Containerized frontend and backend apps
- CI/CD with GitHub Actions
- Ansible provisioning of Ubuntu VM with Docker
- Deployment via Docker Compose
- Watchtower monitoring and auto-updating containers
- (Bonus) K3s + ArgoCD for GitOps on Kubernetes

---

## Requirements

- Ubuntu 22.04 VM (Azure or local)
- Docker & Docker Compose
- GitHub Actions (CI/CD)
- Ansible (provisioning)
- (Optional) K3s + ArgoCD

---

## Steps to test the application

1. Clone this repo:
   ```
   git clone https://github.com/tahaaa22/devops-task.git
   cd devops-task

2. Run ansible (you need to create inventory.ini setup-docker.yml):
   make sure you are in your local machine. the local machine will automate actions in your VMs using ansible

   ```
   ansible-playbook -i inventory.ini setup-docker.yml
   ```

---
3. Docker compose is active and the watchtower is on for 30 sec/interval

4. Test the watchtower
   ```
   echo "// Dummy update to trigger new image" >> README.md
   docker build -t tahaaa894/silver-task:latest .
   docker push tahaaa894/silver-task:latest
   ```
5. SSH into the VM and monitor the watchtower logs
   ```
     sudo docker logs -f $(sudo docker ps -qf "ancestor=containrrr/watchtower")
   ```
   Within 30 seconds, the output will be:
   ```
     Found new silver-task:latest image
    Stopping /silver-task (old container)
    Creating /silver-task (new container)
   ```

## GitHub Actions

GitHub Actions is used for continuous integration and delivery.

* On push to `master`, GitHub Actions:

  * Builds Docker images
  * Pushes them to Docker Hub
  * Triggers deployment using Ansible via SSH

Workflow defined in `.github/workflows/docker-image.yml`.

---

## 📦 Ansible Setup

1. Create or edit the `inventory.ini` file:

   ```ini
   [web]
   4.232.134.109 ansible_user=azureuser ansible_ssh_private_key_file=~/.ssh/LockedIn-VM_key.pem
   ```

2. Run the playbook:

   ```bash
   ansible-playbook -i inventory.ini ansible/deploy.yml
   ```

The playbook will:

* Install Docker
* Copy `docker-compose.yml` to the VM
* Deploy containers using Docker Compose

---

## ⏱ Watchtower

Watchtower is configured in `docker-compose.yml` to monitor and auto-update running containers:

```yaml
  watchtower:
    image: containrrr/watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 30
```

---

## ☸️ Bonus: Kubernetes + ArgoCD

### 🔧 Install K3s

```bash
curl -sfL https://get.k3s.io | sh -
```

### 🚀 Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 🔐 Get ArgoCD Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 🌐 Access ArgoCD Dashboard (I used port forward, but for best practice we can switch to proxy server)

```bash
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Then open: [https://localhost:8081](https://localhost:8081)

### 📦 Deploy App via ArgoCD

```bash
argocd login localhost:8081
argocd app create todo-app \
  --repo https://github.com/tahaaa22/devops-task \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
argocd app sync todo-app
```

---

