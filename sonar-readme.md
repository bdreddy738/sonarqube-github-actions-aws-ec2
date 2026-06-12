# SONARREADME.md

# SonarQube Integration with GitHub Actions on AWS EC2 (Amazon Linux 2023)

## Project Overview

This document describes the complete setup of SonarQube, Sonar Scanner, GitHub Actions integration, GitHub Secrets configuration, and automated code quality analysis for the AIOps project.

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
    v
SonarQube Scanner
    |
    v
SonarQube Server (AWS EC2)
    |
    v
Quality Gate / Code Analysis
```

---

# Environment Details

| Component  | Value                   |
| ---------- | ----------------------- |
| OS         | Amazon Linux 2023       |
| Java       | Amazon Corretto/OpenJDK |
| SonarQube  | Community Edition       |
| Repository | aiops-mlops-devops-labs |
| Branch     | main                    |
| CI/CD      | GitHub Actions          |

---

# Step 1: Connect to EC2

```bash
ssh -i key.pem ec2-user@<PUBLIC-IP>
```

Become root:

```bash
sudo su -
```

---

# Step 2: Verify Java Installation

Check Java:

```bash
java -version
```

Example Output:

```text
openjdk version "26.0.1"
OpenJDK Runtime Environment Corretto
```

If Java is not installed:

```bash
sudo dnf install java-17-amazon-corretto -y
```

Verify:

```bash
java -version
```

---

# Step 3: Download SonarQube

Move to opt directory:

```bash
cd /opt
```

Download SonarQube:

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.6.0.109173.zip
```

Install unzip:

```bash
dnf install unzip -y
```

Extract:

```bash
unzip sonarqube-25.6.0.109173.zip
```

Rename:

```bash
mv sonarqube-25.6.0.109173 sonarqube
```

---

# Step 4: Create SonarQube User

```bash
useradd sonar
```

Assign ownership:

```bash
chown -R sonar:sonar /opt/sonarqube
```

Verify:

```bash
ls -ld /opt/sonarqube
```

---

# Step 5: Start SonarQube

Switch user:

```bash
su - sonar
```

Navigate:

```bash
cd /opt/sonarqube/bin/linux-x86-64
```

Start:

```bash
./sonar.sh start
```

Verify:

```bash
ps -ef | grep sonar
```

Expected:

```text
java
elasticsearch
sonarqube
```

---

# Step 6: Open SonarQube

Browser:

```text
http://<PUBLIC-IP>:9000
```

Example:

```text
http://98.93.100.67:9000
```

Default Credentials:

```text
Username : admin
Password : admin
```

Change password after first login.

---

# Step 7: Create SonarQube Project

Navigate:

```text
Projects
→ Create Project
```

Create:

```text
Project Key  : demo-project
Project Name : demo-project
```

Save.

---

# Step 8: Generate SonarQube Token

Navigate:

```text
My Account
→ Security
→ Generate Token
```

Example:

```text
Name : github-token
```

Generated Token:

```text
squ_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Save token immediately.

---

# Step 9: Install Git

```bash
sudo dnf install git -y
```

Verify:

```bash
git --version
```

---

# Step 10: Clone Repository

```bash
cd /opt

git clone https://github.com/bdreddy738/aiops-mlops-devops-labs.git
```

Navigate:

```bash
cd aiops-mlops-devops-labs
```

---

# Step 11: Install Sonar Scanner

Navigate:

```bash
cd /opt
```

Download:

```bash
curl -L -o sonar-scanner.zip https://repo.maven.apache.org/maven2/org/sonarsource/scanner/cli/sonar-scanner-cli/7.0.2.4839/sonar-scanner-cli-7.0.2.4839-linux-x64.zip
```

Extract:

```bash
unzip sonar-scanner.zip
```

Rename:

```bash
mv sonar-scanner-7.0.2.4839-linux-x64 sonar-scanner
```

Add PATH:

```bash
echo 'export PATH=$PATH:/opt/sonar-scanner/bin' >> ~/.bashrc
```

Load profile:

```bash
source ~/.bashrc
```

Verify:

```bash
sonar-scanner --version
```

Expected:

```text
SonarScanner CLI
Java
Linux
```

---

# Step 12: Create sonar-project.properties

Navigate:

```bash
cd /opt/aiops-mlops-devops-labs
```

Create file:

```bash
vi sonar-project.properties
```

Contents:

```properties
sonar.projectKey=demo-project
sonar.projectName=demo-project
sonar.sources=DAY-01/aiops-lab
sonar.host.url=http://98.93.100.67:9000
sonar.token=squ_xxxxxxxxxxxxxxxxxxxxxxxxx
```

Save file.

---

# Step 13: Manual Sonar Scan

Run:

```bash
sonar-scanner
```

Expected:

```text
ANALYSIS SUCCESSFUL
EXECUTION SUCCESS
```

---

# Step 14: Configure GitHub Secrets

Navigate:

```text
Repository
→ Settings
→ Secrets and variables
→ Actions
```

Create Secret 1:

```text
Name  : SONAR_TOKEN
Value : squ_xxxxxxxxxxxxxxxxxxxxx
```

Create Secret 2:

```text
Name  : SONAR_HOST_URL
Value : http://98.93.100.67:9000
```

Important:

Store actual values.

Do NOT store:

```text
${{ secrets.SONAR_TOKEN }}
${{ secrets.SONAR_HOST_URL }}
```

---

# Step 15: Create GitHub Workflow

Create:

```text
.github/workflows/sonarqube.yml
```

Contents:

```yaml
name: SonarQube Scan

on:
  push:
    branches:
      - main

jobs:
  sonar:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v6
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

---

# Step 16: Push Workflow

```bash
git add .

git commit -m "Added SonarQube integration"

git push origin main
```

---

# Step 17: Trigger Scan Without Code Changes

```bash
git commit --allow-empty -m "Trigger SonarQube scan"

git push origin main
```

Purpose:

```text
Trigger GitHub Actions
Trigger SonarQube Analysis
No source code changes required
```

---

# Step 18: Verify GitHub Actions

Navigate:

```text
GitHub Repository
→ Actions
```

Expected:

```text
SonarQube Scan
✓ Success
```

---

# Step 19: Verify SonarQube Dashboard

Navigate:

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
Coverage
Duplications
```

Expected:

```text
Quality Gate : Passed
Security     : A
Reliability  : A
Maintainability : A
```

---

# Common Troubleshooting

## SonarQube Not Starting

Check:

```bash
ps -ef | grep sonar
```

---

## Port 9000 Not Reachable

Verify AWS Security Group:

```text
TCP 9000
Source 0.0.0.0/0
```

---

## Invalid Token

Generate new token:

```text
My Account
→ Security
→ Generate Token
```

Update:

```text
SONAR_TOKEN
```

GitHub Secret.

---

## Project Not Found

Verify:

```properties
sonar.projectKey=demo-project
```

Matches SonarQube project key.

---

## Invalid Source Directory

Verify:

```properties
sonar.sources=DAY-01/aiops-lab
```

Directory exists in repository.

---
# Step 11A: Configure sonar-scanner.properties

After installing Sonar Scanner, verify the global scanner configuration file.

Location:

```bash
/opt/sonar-scanner/conf/sonar-scanner.properties
```

Open file:

```bash
vi /opt/sonar-scanner/conf/sonar-scanner.properties
```

Default configuration:

```properties
#----- Default SonarQube server
#sonar.host.url=http://localhost:9000
```

Update as needed:

```properties
sonar.host.url=http://98.93.100.67:9000
```

Save and exit.

Verify configuration:

```bash
cat /opt/sonar-scanner/conf/sonar-scanner.properties
```

Purpose:

* Global Sonar Scanner configuration
* Defines default SonarQube server URL
* Used when executing manual scans from EC2
* Applies to all projects scanned from this scanner installation

Example manual scan flow:

```bash
cd /opt/aiops-mlops-devops-labs

sonar-scanner
```

Difference between configuration files:

| File                     | Purpose               | Scope               |
| ------------------------ | --------------------- | ------------------- |
| sonar-scanner.properties | Scanner configuration | Global              |
| sonar-project.properties | Project configuration | Repository Specific |

Global Configuration:

```text
/opt/sonar-scanner/conf/sonar-scanner.properties
```

Project Configuration:

```text
aiops-mlops-devops-labs/sonar-project.properties
```
https://docs.sonarsource.com/sonarqube-server/9.8/try-out-sonarqube

# Final Result

✅ SonarQube Installed

✅ SonarQube Running on AWS EC2

✅ Sonar Scanner Installed

✅ Project Created

✅ Token Generated

✅ Manual Scan Successful

✅ GitHub Secrets Configured

✅ GitHub Actions Integrated

✅ SonarQube Scan Action v6 Configured

✅ Automated Code Quality Analysis Enabled

✅ Quality Gate Validation Working

✅ End-to-End DevSecOps Integration Completed
