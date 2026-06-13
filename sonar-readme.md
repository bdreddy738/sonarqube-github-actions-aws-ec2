# SonarQube + Trivy + Docker Integration with GitHub Actions on AWS EC2 (Amazon Linux 2023)

## Project Overview

This project demonstrates an end-to-end DevSecOps pipeline using:

* SonarQube for Code Quality Analysis
* Trivy for Vulnerability Scanning
* Docker for Containerization
* GitHub Actions for CI/CD Automation
* AWS EC2 for SonarQube Hosting

---

# Architecture

```text
Developer
    |
    v
GitHub Repository
    |
    v
GitHub Actions
    |
    +--> Trivy Filesystem Scan
    |
    +--> Docker Build
    |
    +--> Trivy Image Scan
    |
    +--> SonarQube Scan
    |
    +--> Quality Gate Validation
    |
    v
SonarQube Server (AWS EC2)
```

---

# Environment Details

| Component        | Value                            |
| ---------------- | -------------------------------- |
| OS               | Amazon Linux 2023                |
| Java             | Amazon Corretto 17               |
| SonarQube        | Community Edition                |
| Repository       | sonarqube-github-actions-aws-ec2 |
| Branch           | main                             |
| CI/CD            | GitHub Actions                   |
| Security Scanner | Trivy                            |
| Containerization | Docker                           |

---

# Step 1: Connect to EC2

AWS Console:

```text
AWS Console
→ EC2
→ Instances
→ Select Instance
→ Connect
→ EC2 Instance Connect
→ Connect
```

Become root:

```bash
sudo su -
```

Verify:

```bash
whoami
```

Expected:

```text
root
```

---

# Step 2: Update Server

```bash
dnf update -y
```

---

# Step 3: Install Java

```bash
dnf install java-17-amazon-corretto -y

java -version
```

---

# Step 4: Install Required Packages

```bash
dnf install wget unzip git -y
```

---

# Step 5: Configure Amazon Linux 2023 Kernel Settings

```bash
echo "vm.max_map_count=524288" >> /etc/sysctl.conf

echo "fs.file-max=131072" >> /etc/sysctl.conf

sysctl -p
```

Configure limits:

```bash
echo "sonar   -   nofile   131072" >> /etc/security/limits.conf

echo "sonar   -   nproc    8192" >> /etc/security/limits.conf
```

---

# Step 6: Download SonarQube

```bash
cd /opt

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.6.0.109173.zip

unzip sonarqube-25.6.0.109173.zip

mv sonarqube-25.6.0.109173 sonarqube
```

---

# Step 7: Create SonarQube User

```bash
useradd sonar

chown -R sonar:sonar /opt/sonarqube
```

---

# Step 8: Start SonarQube

```bash
su - sonar

cd /opt/sonarqube/bin/linux-x86-64

./sonar.sh start
```

Verify:

```bash
ps -ef | grep sonar
```

---

# Step 9: Open SonarQube

```text
http://<EC2-PUBLIC-IP>:9000
```

Default Credentials:

```text
Username : admin
Password : admin
```

Change password after first login.

---

# Step 10: Create SonarQube Project

```text
Projects
→ Create Project

Project Key  : demo-project
Project Name : demo-project
```

---

# Step 11: Generate SonarQube Token

```text
My Account
→ Security
→ Generate Token

Name : github-token
```

Save the generated token.

---

# Step 12: Clone Repository

```bash
cd /opt

git clone https://github.com/bdreddy738/sonarqube-github-actions-aws-ec2.git

cd sonarqube-github-actions-aws-ec2
```

---

# Step 13: Install Sonar Scanner

```bash
cd /opt

curl -L -o sonar-scanner.zip https://repo.maven.apache.org/maven2/org/sonarsource/scanner/cli/sonar-scanner-cli/7.0.2.4839/sonar-scanner-cli-7.0.2.4839-linux-x64.zip

unzip sonar-scanner.zip

mv sonar-scanner-7.0.2.4839-linux-x64 sonar-scanner
```

Add PATH:

```bash
echo 'export PATH=$PATH:/opt/sonar-scanner/bin' >> ~/.bashrc

source ~/.bashrc
```

Verify:

```bash
sonar-scanner --version
```

---

# Step 14: Configure sonar-scanner.properties

```bash
vi /opt/sonar-scanner/conf/sonar-scanner.properties
```

Add:

```properties
sonar.host.url=http://<EC2-PUBLIC-IP>:9000
```

---

# Step 15: Configure sonar-project.properties

```properties
sonar.projectKey=demo-project
sonar.projectName=demo-project
sonar.sources=.
```

---

# Step 16: Configure GitHub Secrets

GitHub Repository:

```text
Settings
→ Secrets and Variables
→ Actions
```

Create:

```text
SONAR_TOKEN=<generated-token>

SONAR_HOST_URL=http://<EC2-PUBLIC-IP>:9000
```

---

# Step 17: GitHub Actions Workflow

Pipeline Flow:

```text
Checkout Code
      ↓
Trivy Filesystem Scan
      ↓
Docker Build
      ↓
Trivy Image Scan
      ↓
Setup Java
      ↓
SonarQube Scan
      ↓
Quality Gate Validation
```

---

# Step 18: Push Changes

```bash
git add . && git commit -m "Added DevSecOps pipeline with Trivy, Docker and SonarQube" && git push origin main
```

---

# Step 19: Trigger Pipeline Again

```bash
git commit --allow-empty -m "Trigger DevSecOps pipeline"

git push origin main
```

---

# Step 20: Verify GitHub Actions

```text
Repository
→ Actions
```

Expected:

```text
✓ Trivy Filesystem Scan

✓ Docker Build

✓ Trivy Image Scan

✓ SonarQube Scan

✓ Quality Gate Check
```

---

# Step 21: Verify SonarQube Dashboard

```text
Projects
→ demo-project
```

Verify:

```text
Quality Gate
Security Rating
Reliability Rating
Maintainability Rating
Code Smells
Bugs
Vulnerabilities
```

---

# AWS Security Group

```text
SSH             22      My IP

Custom TCP      9000    0.0.0.0/0
```

---

# Repository Structure

```text
sonarqube-github-actions-aws-ec2/
│
├── .github/
│   └── workflows/
│       └── sonarqube.yml
│
├── Dockerfile
├── index.html
├── sonar-project.properties
├── README.md
└── SONARREADME.md
```

---

# Final Result

✅ SonarQube Installed

✅ Sonar Scanner Installed

✅ SonarQube Running on AWS EC2

✅ Docker Build Enabled

✅ Trivy Filesystem Scan Enabled

✅ Trivy Image Scan Enabled

✅ GitHub Actions Integrated

✅ GitHub Secrets Configured

✅ SonarQube Quality Gate Validation Working

✅ End-to-End DevSecOps Pipeline Completed
