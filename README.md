# Project Overview

![Project Architecture](3-Tier-Architecture.jpeg)

This project aims to deploy a scalable, highly available, and secure Java application using a 3-tier Amazon Web Services (AWS) architecture. The architecture includes a Virtual Private Cloud (VPC) setup with separate networks for a Bastion Host, application servers, and essential components like Nginx, Tomcat, and a MySQL database.

## Architecture Highlights

- **VPC Setup:** Establishes a robust network structure, including public and private subnets, to ensure security and efficient communication.
  
- **Bastion Host:** Facilitates secure access to the private network for administration purposes.

- **Maven Build:** Implements a continuous integration and continuous deployment (CI/CD) pipeline using Maven, Bitbucket, Sonarcloud, and Jfrog for efficient code management and deployment.

- **3-Tier Architecture:** Deploys a MySQL RDS database as the backend, a Tomcat application server as the middleware, and Nginx as the frontend, all orchestrated in a highly available and auto-scalable setup.

- **Post-Deployment Actions:** Configures cron jobs, Cloudwatch alarms, and other post-deployment tasks to ensure optimal system performance and monitoring.

## Purpose

The primary goal of this project is to showcase best practices in deploying and managing a Java application in a cloud environment. It demonstrates the use of various AWS services, automation tools, and CI/CD practices to achieve a secure, scalable, and highly available architecture. Developers and cloud enthusiasts can leverage this project to understand and implement similar setups for their applications.

Here's your content converted into a well-structured **Markdown (.md)** file format:

```markdown
# Goal

The goal of this project is to deploy a scalable, highly available, and secured Java application on a 3-tier architecture and provide application access to end users from the public internet.

---

# Pre-Requisites

- Create an AWS Free Tier account  
- Create a Bitbucket account and a repository to store Java source code  
- Migrate Java Source Code to your own Bitbucket repository – Refer to documentation  
- Create an account in [SonarCloud](https://sonarcloud.io/)  
- Create an account in [JFrog Cloud](https://jfrog.com/)

---

# Pre-Deployment

### Create Global AMI

- AWS CLI  
- CloudWatch agent  
- Install AWS SSM agent

### Create Golden AMIs

#### Nginx Application AMI

- Install Nginx  
- Push custom memory metrics to CloudWatch

#### Apache Tomcat Application AMI

- Install Apache Tomcat  
- Configure Tomcat as a systemd service  
- Install JDK 11  
- Push custom memory metrics to CloudWatch

#### Apache Maven Build Tool AMI

- Install Apache Maven  
- Install Git  
- Install JDK 11  
- Update Maven Home in system `PATH` environment variable

---

# VPC Deployment

Deploy AWS Infrastructure resources as per the architecture.

## VPC (Network Setup)

- Create VPC network `192.168.0.0/16` for **Bastion Host**
- Create VPC network `172.32.0.0/16` for **application servers**
- Create **NAT Gateway** in public subnet and update private subnet route table
- Create **Transit Gateway** and associate both VPCs for private communication
- Create **Internet Gateway** for each VPC and update public subnet route table

---

# Bastion Host

- Deploy Bastion Host in Public Subnet with **EIP**
- Create **Security Group** allowing port `22` from the public internet

---

# Maven (Build Server)

1. Create EC2 instance using **Maven Golden AMI**
2. Clone Bitbucket repository into VSCode
3. Update `pom.xml` with Sonar and JFrog deployment details
4. Add `settings.xml` with JFrog credentials and repo settings
5. Update `application.properties` with JDBC connection string (MySQL)
6. Push code changes to **feature branch**
7. Raise **Pull Request** and merge into **master**
8. SSH into EC2 instance and clone Bitbucket repository
9. Build source using Maven:  
```

mvn clean install -s settings.xml

```
10. Integrate with SonarCloud and verify Quality Gate

---

# 3-Tier Architecture

## Database Tier (RDS)

- Deploy **Multi-AZ MySQL RDS** in private subnets
- Create **Security Group** allowing port `3306` from:
- App instances  
- Bastion Host

## Application Tier (Tomcat Backend)

- Create **private Network Load Balancer** and **Target Group**
- Create **Launch Configuration** with:
- Tomcat Golden AMI  
- User Data script to deploy `.war` from JFrog  
- Security Group allowing:
 - Port `22` from Bastion  
 - Port `8080` from private NLB
- Create **Auto Scaling Group**

## Web Tier (Nginx Frontend)

- Create **public Network Load Balancer** and **Target Group**
- Create **Launch Configuration** with:
- Nginx Golden AMI  
- User Data script to update `nginx.conf` proxy_pass and reload service  
- Security Group allowing:
 - Port `22` from Bastion  
 - Port `80` from public NLB
- Create **Auto Scaling Group**

---

# Application Deployment

- Artifact deployment is handled by **User Data** during EC2 instance launch.
- SSH into Application Server, login to MySQL via CLI:
- Create database and table schema  
- (Refer to `README.md` in Bitbucket repo for exact commands)

---

# Post-Deployment

- Configure **Cronjob** to:
- Push Tomcat logs to **S3 bucket**
- Rotate and remove logs locally after upload

- Configure **CloudWatch Alarms** to:
- Send email when DB connections exceed 100

---

# Validation

- ✅ Verify **Admin access** via:
- Session Manager  
- Bastion Host

- ✅ Verify **User access** via:
- Public browser access to application

```

You can copy this and save it as `README.md` or any `.md` file.

Would you like me to generate an `.md` file or a downloadable version as well?





