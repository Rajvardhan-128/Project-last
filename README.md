## 🚀 Jenkins–Nexus–SonarQube–Terraform–EKS End-to-End DevOps Automation
📘 Overview

This project demonstrates a complete DevOps pipeline that automates infrastructure provisioning, CI/CD setup, security scanning, artifact management, containerization, and deployment onto an AWS EKS cluster.

The workflow is built using Terraform, Jenkins, Nexus, SonarQube, Docker, and AWS EKS — with integrations configured through real pipelines and credentials.

🧩 Repositories Involved - 

Repository	Purpose

🔗 Jenkins-Nexus-Sonar  :

Automates provisioning of Jenkins, Nexus, and SonarQube servers using Terraform  

🔗 Project-Last (EKS Deployment)  :

Jenkins pipeline to create AWS EKS Cluster and Kubernetes configuration  
	
🔗 Ekart  :

Java-based application integrated with Maven, Nexus, SonarQube, Docker, and EKS for CI/CD deployment
	
## ⚙️ Project Workflow - 

## Phase 1: Infrastructure Provisioning using Terraform

Repository: Jenkins-Nexus-Sonar

🔧 Steps:

1. Create AWS Network Infrastructure.

		Define main.tf for VPC, Internet Gateway (IGW), Subnets (2 public + 2 private), Route Tables, and Security Groups.

2. Associate each subnet with its route table and configure inbound rules for required ports.

3. Provision Jenkins Server.

	- Define jenkins.tf for a t3.medium instance (Ubuntu) with 30GB EBS volume.

	- Use user-data from jenkins-server.sh to install Jenkins, Java, Docker, and initial setup.

4. Provision Nexus and SonarQube Servers

	- Repeat the same structure (nexus.tf and sonar.tf) using custom setup scripts to install and configure each service via Docker.

5. IAM Role Configuration

	- Create IAM roles and policies for access management and EC2 permissions.

6. Define Variable and Output Files

	- variables.tf, provider.tf, outputs.tf, and data.tf manage dynamic values and resource outputs.

7.Initialize and Apply Terraform

	terraform init
	terraform plan
	terraform apply


- Ensure the key pair is available in your AWS account before applying.

## Post-Provision Setup :

- Login to each server after deployment:

9. Jenkins: Complete portal setup.

10. Nexus: Retrieve admin password via:

		docker ps
		docker exec -it <container_id> /bin/bash
		cd /nexus-data/
		cat admin.password


11. SonarQube:

    - Default login → admin / admin, then reset password.

## Phase 2: Jenkins Configuration and Integrations

🔌 Install Required Plugins  

| Plugin                     | Purpose                        |
|----------------------------|--------------------------------|
| SonarQube Scanner           | Code quality analysis          |
| Nexus Artifact Uploader     | Upload artifacts to Nexus      |
| Docker, Docker Pipeline     | Image build and push           |
| OWASP Dependency-Check      | Vulnerability scanning         |
| Eclipse Temurin Installer   | Install JDK                    |
| Pipeline Maven Integration  | CI/CD build stages             |

⚙️ Configure Tools

1. JDK:
   
	- Jenkins → Manage Jenkins → Tools

	- Add JDK → Install from adoptium.net → Version jdk-17.0.9+9

2. Sonar Scanner:

	- Default version, automatic install.

3. Maven:

	- Name: maven3, Version: 3.6.3, Path: /usr/share/maven

4. Dependency Check:

 - Name: DC, Version: 6.5.1

5. Docker:

- Latest version (from Docker.io)

## Phase 3: Credentials & Integrations

🔑 Credentials Setup  

| Tool                | Type        | ID            | Description                                 |
|--------------------|------------|---------------|---------------------------------------------|
| SonarQube Token     | Secret Text | sonar-quabe   | Generated via Sonar → Security → Users     |
| Docker Hub Password | Secret Text | dockerhub-pwd | Docker Hub credentials                      |
| NVD API Key         | Secret Text | nvd-api-key   | Used in OWASP Dependency-Check             |

🔗 Tool Integrations

1. SonarQube Integration - 
   
- Jenkins → Manage Jenkins → System → SonarQube Installations

		Name: sonar-token

		Server URL: SonarQube URL

		Token: sonar-quabe

2. Nexus Integration - 
   
- Jenkins → Manage Jenkins → Managed Files → Add new config (global-maven)

- Add following lines:

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

## Phase 4: EKS Cluster Deployment Pipeline

- Repository: Project-last

🧩 Steps:

1.Create a Jenkins pipeline named “EKS-Cluster-Deployment”.

- Configure SCM:

		Repository: https://github.com/Rajvardhan-128/Project-last.git
	
		Branch: main

2. Run the pipeline to automatically:

3. Provision AWS EKS cluster using Terraform.

- Create 2 worker nodes.

- Configure kubectl access to Jenkins.

## Phase 5: Ekart Application CI/CD Deployment

- Repository: Ekart

🧱 Steps:

1. Create Jenkins pipeline: “Deployment”

  		SCM → Git → https://github.com/Rajvardhan-128/Ekart.git

        Branch: main

2. The pipeline stages include:

1. Checkout SCM
2. Tool Install
3. Git Checkout
4. OWASP Dependency Check
5. Compile
6. Unit Test
7. SonarQube Analysis
8. Build
9. Deploy to Nexus
10. Build and Tag Docker Image
11. Push Image to DockerHub
12. Deploy to EKS via kubectl
13. Ensure the settings.xml file is configured inside:  

        /var/lib/jenkins/.m2/settings.xml

- code inside that file is below :

		 <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
	          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
	                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
	
	  <!-- Maven local repository location -->
	  <localRepository>/var/lib/jenkins/.m2/repository</localRepository>
	
	  <!-- Default global mirrors -->
	  <mirrors>
	    <mirror>
	      <id>central</id>
	      <mirrorOf>central</mirrorOf>
	      <url>https://repo.maven.apache.org/maven2</url>
	    </mirror>
	  </mirrors>
	
	  <!-- Default profiles -->
	  <profiles>
	    <profile>
	      <id>default</id>
	      <repositories>
	        <repository>
	          <id>central</id>
	          <url>https://repo.maven.apache.org/maven2</url>
	        </repository>
	      </repositories>
	    </profile>
	  </profiles>
	
	  <!-- Activate the default profile -->
	  <activeProfiles>
	    <activeProfile>default</activeProfile>
	  </activeProfiles>
	
	</settings>


🧠 Issues Faced & Solutions

Issue	Description	Solution : 
- NVD API Key Error	OWASP Dependency Check failed due to missing API key.	Added nvd-api-key as secret text in Jenkins credentials.
- Compile-Time Error	Maven build failed due to version mismatch.	Fixed by updating JDK and Maven versions in Jenkins tool config.
- SonarQube Connection Failure	Jenkins could not connect to Sonar server.	Added correct server URL and token in SonarQube plugin configuration.
- Docker Permission Denied	Jenkins user lacked Docker access.	Added jenkins user to docker group and restarted the service.

📸 (images of Jenkins pipeline execution and issues here for visual demonstration.)
<img src="/images/image1.png">
(./images/image1.png)
(./images/image2.png)
(./images/image3.png)
(./images/image4.png)

✅ Final Pipeline Execution

- The final pipeline runs seamlessly with all green stages:
(./images/image5.png)

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
