# CI/CD Pipeline Setup using GitHub Actions, SonarQube, Nexus & Tomcat on AWS EC2

This guide walks through setting up a complete CI/CD pipeline using GitHub Actions with SonarQube for code quality, Nexus for artifact storage, and Tomcat for deployment, all hosted on AWS EC2 instances.

### Step 1: Launch EC2 Instance for SonarQube

Purpose: Code Quality & Static Analysis

Go to AWS EC2 Console → Launch Instance

Select Ubuntu 22.04 LTS as AMI

Choose an instance type (Recommended: t2.medium or higher)

Select your Key Pair

Configure security group → allow Port 9000

Launch the instance

<img width="1919" height="860" alt="bd3d0e37bb79e26952ec8b6ad62c02d2_Screenshot%202025-11-10%20114032" src="https://github.com/user-attachments/assets/0769511d-fcc8-4a50-be43-cb6063bfe919" />


### Step 2: Connect to Instance & Install Java

Copy SSH command from AWS console

Connect from your terminal:

ssh -i <keypair.pem> ubuntu@<EC2-Public-IP>

Install the latest version of Java:

sudo apt update
sudo apt install openjdk-17-jdk -y
java -version

<img width="1470" height="757" alt="b90baf362075adc011e70ffe90bcbb19_Screenshot%202025-11-10%20114154" src="https://github.com/user-attachments/assets/d1209402-d72f-4a63-89c8-cf705114d580" />


### Step 3: Install & Start SonarQube

Download and install SonarQube:

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.87278.zip

unzip sonarqube-10.4.1.87278.zip

cd sonarqube-10.4.1.87278/bin/linux-x86-64

./sonar.sh start

<img width="1474" height="755" alt="5764be7edd8ce320afc70f107bb9292f_Screenshot%202025-11-10%20114702" src="https://github.com/user-attachments/assets/0c0e98be-6e69-4834-b9eb-586f287f5033" />

##### Verify service is running:

./sonar.sh status

<img width="1470" height="756" alt="c3b75c74cfec32303a9bc50e40a43223_Screenshot%202025-11-10%20115011" src="https://github.com/user-attachments/assets/077d7391-93b3-4a3e-9497-71035f2cf788" />

#### Access SonarQube in browser:

 http://<EC2-Public-IP>:9000

Default credentials: admin / admin

Generate a Sonar Token for GitHub Actions.

<img width="1919" height="961" alt="6a22fcf5878ed08740df2a74e82f12f1_Screenshot%202025-11-10%20115206" src="https://github.com/user-attachments/assets/f4aba40a-1a12-4663-bdd3-522a48ab5698" />

### Step 4: Launch EC2 Instance for Nexus

Purpose: Artifact Repository

Go to AWS EC2 Console → Launch Instance

Select Ubuntu 22.04 LTS as AMI

Choose an instance type 

Select your Key Pair

Allow Port 8081

<img width="1906" height="784" alt="4931b516ce25a24d4dac61dbcb597929_Screenshot%202025-11-10%20115718" src="https://github.com/user-attachments/assets/501e2841-a70e-420f-b8e2-22ca67b8cc61" />

##### SSH into the instance and install Nexus:

wget https://download.sonatype.com/nexus/3/nexus-3.85.0-03-linux-x86_64.tar.gz

tar -xvf nexus-3.85.0-03-linux-x86_64.tar.gz

mv nexus-3.85.0-03 nexus

./nexus/bin/nexus start

<img width="1467" height="701" alt="2d2a7a85a9307a76b4ce0437b17545e6_Screenshot%202025-11-11%20103556" src="https://github.com/user-attachments/assets/5445fded-03f0-44ed-afdf-e06534f4f7c5" />

#### Access Nexus:

 http://<EC2-Public-IP>:8081

<img width="1919" height="909" alt="1f07d32dfe889238bc3c3d0e51df8d1f_Screenshot%202025-11-10%20121609" src="https://github.com/user-attachments/assets/3b93994a-be3d-40a9-936a-9e75a35679de" />


### Step 5: Launch EC2 Instance for Tomcat

Purpose: Deployment Server

Go to AWS EC2 Console → Launch Instance

Select Ubuntu 22.04 LTS as AMI

Choose an instance type

Select your Key Pair

<img width="1900" height="780" alt="184f6e2ac9ae5cbc90f9777dd1306047_Screenshot%202025-11-10%20122814" src="https://github.com/user-attachments/assets/14b70a8e-54f9-4081-b6f2-7507f6a5f09e" />

##### SSH into instance and install Tomcat:

wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.112/bin/apache-tomcat-9.0.112.tar.gz

tar -xvzf apache-tomcat-9.0.112.tar.gz

cd apache-tomcat-9.0.112/bin

./startup.sh

<img width="1462" height="748" alt="cc2e7453551931caed70f677a0c723cd_Screenshot%202025-11-11%20103615" src="https://github.com/user-attachments/assets/dfd26626-2dc6-49c1-a602-25855bbde256" />

##### Configure a Tomcat Manager User:

Edit /etc/tomcat9/tomcat-users.xml and add:

<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="admin123" roles="manager-gui,manager-script"/>

#### Access Tomact:

<img width="1299" height="611" alt="36d52160cfe84e98b7ffe7d7e8f666e2_Screenshot%202025-11-11%20121654" src="https://github.com/user-attachments/assets/db0d1a64-a224-40c3-a9d9-ae7f5886c461" />

### Step 6: Configure GitHub Secrets

Go to GitHub Repository → Settings → Secrets and Variables → Actions

Click New Repository Secret and add the following:

Secret Name	Description

SONAR_HOST_URL	SonarQube Server URL

SONAR_TOKEN	Generated Token

NEXUS_USER	Nexus Username

NEXUS_PASS	Nexus Password

TOMCAT_USER	Tomcat Username

TOMCAT_PASS	Tomcat Password

<img width="1910" height="955" alt="0b48c8b595449541085a683573e87db9_Screenshot%202025-11-10%20141358" src="https://github.com/user-attachments/assets/41fdaf99-c502-4b8f-8fa8-7afeeeee79f3" />


### Step 7: Create GitHub Actions Workflow

In your repository, create a new file:

.github/workflows/ci-cd.yml

Paste your pipeline YAML configuration and commit changes.

Navigate to the Actions tab — your CI/CD pipeline will start automatically on a new push to main.

<img width="1887" height="908" alt="d7f4d2a25e950d11475bc3b04af93094_Screenshot%202025-11-11%20103139" src="https://github.com/user-attachments/assets/93953122-88df-415c-9b8a-60104af9d5a0" />
<img width="1919" height="829" alt="8922b8abd86185ad3ef6dc4d1efd744a_Screenshot%202025-11-11%20123241" src="https://github.com/user-attachments/assets/7e7674ab-31c4-4f32-9a2a-d5e2871e4cb9" />


### Step 8: Verify Deployment

After successful execution:

SonarQube: Code Analysis Report

Nexus: Stored Build Artifacts

Tomcat: Deployed Application

<img width="1919" height="790" alt="fb44c8a41c24995cf25c592512bf8e37_Screenshot%202025-11-11%20103022" src="https://github.com/user-attachments/assets/7c89c4a3-eda6-4851-9df2-b6bea411ffe9" />
<img width="1919" height="756" alt="40ae50b0aebf16c4798d147d9b9aa428_Screenshot%202025-11-11%20103034" src="https://github.com/user-attachments/assets/bb98b161-61db-4794-a589-bb0b4bc325fd" />
<img width="1919" height="901" alt="0b17a0f8dc2c0bec53abe675b491d02a_Screenshot%202025-11-11%20102918" src="https://github.com/user-attachments/assets/798d8e4c-6d8a-4a73-9613-1335742ea6b1" />
<img width="1919" height="897" alt="212316d3056419e71f70c8cb331afef4_Screenshot%202025-11-11%20102951" src="https://github.com/user-attachments/assets/3ea56e77-fbef-490d-b960-d42b4124ded2" />


### Final Outcome

A fully automated CI/CD pipeline that:

Analyzes code with SonarQube

Stores artifacts in Nexus

Deploys to Tomcat automatically via GitHub Actions
