🚀 Jenkins–Nexus–SonarQube–Terraform–EKS End-to-End DevOps Automation
📘 Overview

This project demonstrates a complete DevOps pipeline that automates infrastructure provisioning, CI/CD setup, security scanning, artifact management, containerization, and deployment onto an AWS EKS cluster.

The workflow is built using Terraform, Jenkins, Nexus, SonarQube, Docker, and AWS EKS — with integrations configured through real pipelines and credentials.

🧩 Repositories Involved
Repository	Purpose
🔗 Jenkins-Nexus-Sonar
	Automates provisioning of Jenkins, Nexus, and SonarQube servers using Terraform
🔗 Project-Last (EKS Deployment)
	Jenkins pipeline to create AWS EKS Cluster and Kubernetes configuration
🔗 Ekart
	Java-based application integrated with Maven, Nexus, SonarQube, Docker, and EKS for CI/CD deployment
⚙️ Project Workflow
Phase 1: Infrastructure Provisioning using Terraform

Repository: Jenkins-Nexus-Sonar

🔧 Steps:

Create AWS Network Infrastructure

Define main.tf for VPC, Internet Gateway (IGW), Subnets (2 public + 2 private), Route Tables, and Security Groups.

Associate each subnet with its route table and configure inbound rules for required ports.

Provision Jenkins Server

Define jenkins.tf for a t3.medium instance (Ubuntu) with 30GB EBS volume.

Use user-data from jenkins-server.sh to install Jenkins, Java, Docker, and initial setup.

Provision Nexus and SonarQube Servers

Repeat the same structure (nexus.tf and sonar.tf) using custom setup scripts to install and configure each service via Docker.

IAM Role Configuration

Create IAM roles and policies for access management and EC2 permissions.

Define Variable and Output Files

variables.tf, provider.tf, outputs.tf, and data.tf manage dynamic values and resource outputs.

Initialize and Apply Terraform

terraform init
terraform plan
terraform apply


Ensure the key pair is available in your AWS account before applying.

Post-Provision Setup

Login to each server after deployment:

Jenkins: Complete portal setup.

Nexus: Retrieve admin password via:

docker ps
docker exec -it <container_id> /bin/bash
cd /nexus-data/
cat admin.password


SonarQube: Default login → admin / admin, then reset password.

Phase 2: Jenkins Configuration and Integrations
🔌 Install Required Plugins
Plugin	Purpose
SonarQube Scanner	Code quality analysis
Nexus Artifact Uploader	Upload artifacts to Nexus
Docker, Docker Pipeline	Image build and push
OWASP Dependency-Check	Vulnerability scanning
Eclipse Temurin Installer	Install JDK
Pipeline Maven Integration	CI/CD build stages
⚙️ Configure Tools

JDK:
Jenkins → Manage Jenkins → Tools

Add JDK → Install from adoptium.net → Version jdk-17.0.9+9

Sonar Scanner:

Default version, automatic install.

Maven:

Name: maven3, Version: 3.6.3, Path: /usr/share/maven

Dependency Check:

Name: DC, Version: 6.5.1

Docker:

Latest version (from Docker.io)

Phase 3: Credentials & Integrations
🔑 Credentials Setup
Tool	Type	ID	Description
SonarQube Token	Secret Text	sonar-quabe	Generated via Sonar → Security → Users
Docker Hub Password	Secret Text	dockerhub-pwd	Docker Hub credentials
NVD API Key	Secret Text	nvd-api-key	Used in OWASP Dependency-Check
🔗 Tool Integrations

SonarQube Integration
Jenkins → Manage Jenkins → System → SonarQube Installations

Name: sonar-token

Server URL: SonarQube URL

Token: sonar-quabe

Nexus Integration
Jenkins → Manage Jenkins → Managed Files → Add new config (global-maven)
Add following lines:

<server>
    <id>maven-releases</id>
    <username>admin</username>
    <password>Vishv@1282</password>
</server>
<server>
    <id>maven-snapshots</id>
    <username>admin</username>
    <password>Vishv@1282</password>
</server>

Phase 4: EKS Cluster Deployment Pipeline

Repository: Project-last

🧩 Steps:

Create a Jenkins pipeline named “EKS-Cluster-Deployment”.

Configure SCM:

Repository: https://github.com/Rajvardhan-128/Project-last.git

Branch: main

Run the pipeline to automatically:

Provision AWS EKS cluster using Terraform.

Create 2 worker nodes.

Configure kubectl access to Jenkins.

Phase 5: Ekart Application CI/CD Deployment

Repository: Ekart

🧱 Steps:

Create Jenkins pipeline: “Deployment”

SCM → Git → https://github.com/Rajvardhan-128/Ekart.git

Branch: main

The pipeline stages include:

Checkout SCM

Tool Install

Git Checkout

OWASP Dependency Check

Compile

Unit Test

SonarQube Analysis

Build

Deploy to Nexus

Build and Tag Docker Image

Push Image to DockerHub

Deploy to EKS via kubectl

Ensure the settings.xml file is configured inside:

/var/lib/jenkins/.m2/settings.xml

🧠 Issues Faced & Solutions
Issue	Description	Solution
NVD API Key Error	OWASP Dependency Check failed due to missing API key.	Added nvd-api-key as secret text in Jenkins credentials.
Compile-Time Error	Maven build failed due to version mismatch.	Fixed by updating JDK and Maven versions in Jenkins tool config.
SonarQube Connection Failure	Jenkins could not connect to Sonar server.	Added correct server URL and token in SonarQube plugin configuration.
Docker Permission Denied	Jenkins user lacked Docker access.	Added jenkins user to docker group and restarted the service.

📸 (Add screenshots of Jenkins pipeline execution and issues here for visual demonstration.)

✅ Final Pipeline Execution

The final pipeline runs seamlessly with all green stages:

✔ Checkout SCM →
✔ Compile →
✔ OWASP Dependency Check →
✔ SonarQube Analysis →
✔ Build →
✔ Deploy to Nexus →
✔ Build and Push Docker Image →
✔ Deploy to Kubernetes (EKS)

🧩 Tools Used

Terraform, Jenkins, Nexus, SonarQube, Docker, AWS EKS, Maven, OWASP Dependency-Check, Kubernetes, GitHub
