
# 🚀 Jenkins + Docker + Ansible Setup

This guide walks you through setting up a custom Jenkins container with both Docker and Ansible pre-installed. Ideal for CI/CD pipelines that require Ansible deployments and Docker image builds.

---

## 📦 What's Included

- **Jenkins** 
- **Ansible**
- **Docker (CLI access)** 
- **Custom Dockerfile**
- **Script for easy setup**

---
make sure to change the following variables in the jenkinsfile :
<your-server-ip>
<your-ec2-ip>
<your-dockerhub-username>
<your-image-name>
<your-key>


if you are using ec2 instance, make sure you make elastic ip for your ec2 instance

if you cloning another repository and use this setup in this repo ,
make sure you have the correct files : 
inventory.ini,
deploy-playbook.yml

---

### 1. Create `Dockerfile.custom` file

This file builds a Jenkins container with Docker and Ansible pre-installed:

```Dockerfile
FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y ansible docker.io && \
    apt-get clean

USER jenkins
```

---

### 2. Install and Configure Docker on the Host : 

 Update & Upgrade

```bash
sudo apt update && sudo apt upgrade -y
```

Install Docker

```bash
echo "🛠️ Installing Docker..."
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```
Add current user to docker group
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 3. Build Image :

```bash
sudo docker build -t custom-jenkins:latest -f Dockerfile.custom .
```

### 4. Run Container :

```bash
sudo docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/bin/docker:/usr/bin/docker \
  -u root \
  custom-jenkins:latest
```

---
## 🛡️ Security Groups Setup

- In your EC2 instance, make sure security groups are properly configured to allow traffic on ports 8080 (Jenkins), 50000 (Jenkins), and 22 (SSH). 

---

## ✅ After Setup

- Visit `http://<your-ec2-ip>:8080` to access Jenkins.
- First-time setup password is found in the Jenkins container at 
`/var/jenkins_home/secrets/initialAdminPassword`.

- Get the initial admin password:

```bash
 docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

- Install recommended plugins and configure your first admin user.

---

## 🔐 DockerHub & SSH Credentials

- **DockerHub:** Create credentials in Jenkins (`dockerhub-creds`) for pushing images.
- **SSH Key for EC2:** Add your SSH private key as Jenkins credential (`ec2-ssh-key`) to allow Ansible deployments.

--- 

## in your jenkins open http://<your-ec2-ip>:8080 and follow the instructions to complete the setup.
open "Manage Jenkins" and click "Manage Credentials".
then click "Add Credentials" and add your DockerHub credentials.
add your SSH key for EC2 as a credential.
go to "Manage Jenkins" and click "new item " and add a new job.
in the job configuration, add the following:
use the pipeline project type.
modify pipeline from git repository and add make the build pipeline.

---

## 🔁 Notes

- Jenkins runs as root but switches to `jenkins` user.
- Docker socket is shared to allow containerized builds to access the host daemon.
- Ansible is installed inside Jenkins for provisioning via playbooks.
- make sure you make elastic ip for your ec2 instance 

---
## 📎 Tips

- Add DockerHub and SSH credentials via **Manage Jenkins → Credentials**.
- Use a `Jenkinsfile` to automate Docker build/push and Ansible deployment.
- You can clone this setup to EC2 for production-grade CI/CD.

---
## 🧹 Cleanup

```bash
docker rm -f jenkins
docker rmi custom-jenkins:latest
```

---

# jenkins-simple-pipeline
