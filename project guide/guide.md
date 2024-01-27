# Project Overview

The goal of this project is to deploy a scalable, highly available, and secured Java application on a 3-tier architecture and provide application access to end users from the public internet.

## Pre-Requisites

1. Create an AWS Free Tier account
2. Create a Bitbucket account and a repository to keep your Java source code.
3. Migrate this Java Source Code to your own Bitbucket repository.
4. Create an account in Sonarcloud.
5. Create an account in the Jfrog cloud.

## Pre-Deployment

The instance used to create the custom images can be launched in the default VPC using Amazon Linux 2.

### 1. Create Global AMI

1. **AWS CLI**: Installed by default on Amazon Linux 2 AMI.
2. **CloudWatch agent**:
    ```bash
    sudo su
    yum -y install amazon-cloudwatch-agent
    systemctl status amazon-cloudwatch-agent.service
    ```
3. **Install AWS SSM agent**: Installed by default on Amazon Linux 2 AMI. Verify session manager access by attaching an IAM role with an AWS managed policy named `AmazonSSMFullAccess` (for testing purposes only) and connect from the EC2 AWS console.

### 2. Create Golden AMI using Global AMI for Nginx application

Launch a new instance using the Global AMI created in Task 1 and execute the following steps:

1. **Install Nginx**:
    ```bash
    sudo su
    amazon-linux-extras install -y nginx1
    systemctl start nginx && systemctl enable nginx && systemctl status nginx
    ```
2. **Push custom memory metrics to Cloudwatch**:
   - Pushing custom metrics requires installation and configuration of the CloudWatch agent.
   - Execute the wizard below to configure the CloudWatch agent. Accept most of the defaults, exceptions can include selecting `cwagent` user or selecting the standard default metrics config when prompted.
   ```bash
   /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

3. **Start the CloudWatch the agent specifying the JSON config file**:
  
   ```bash
   /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

4. **Verify the CloudWatch agent is running**:
  
   ```bash
   systemctl status amazon-cloudwatch-agent.service
Once all dependencies are installed, create the AMI as shown below


### 3. Create Golden AMI using Global AMI for the Apache Tomcat application

Launch a new instance using the Global AMI created in Task 1 and execute the following steps:

1. **Install Apache Tomcat**:
    ```bash
    sudo su && cd /opt
    wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.73/bin/apache-tomcat-8.5.73.zip && unzip apache-tomcat-8.5.73.zip
    ```

2. **Configure Tomcat as Systemd service**:
    - Create the unit file with the following contents:
      ```bash
      vi /etc/systemd/system/tomcat.service
      ```
    - Ensure all files are executable in /opt/apache-tomcat-8.5.73/bin
      ```bash
      chmod +x /opt/apache-tomcat-8.5.73/bin/*
      ```
    - Reload all unit config files, start and enable the tomcat service for reboots:
      ```bash
      systemctl daemon-reload && systemctl start tomcat.service && systemctl enable tomcat.service
      ```

3. **Install JDK 11**:
    ```bash
    sudo su && amazon-linux-extras install -y java-openjdk11 && java --version
    ```

4. **Push custom memory metrics to Cloudwatch**:
    - Same steps as shown for nginx in step 2 above
### 4. Create Golden AMI using Global AMI for Apache Maven Build Tool

Launch a new instance using the Global AMI created in Task 1 and execute the following steps:

1. **Install Apache Maven**:
    - Download and unzip the installation package for installation:
      ```bash
      sudo su && cd /opt/ && wget https://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.zip && unzip apache-maven-3.8.4-bin.zip
      ```

2. **Install Git**:
    ```bash
    yum install -y git
    ```

3. **Install JDK 11**:
    ```bash
    amazon-linux-extras install -y java-openjdk11 && java --version
    ```

4. **Update Maven Home to the system PATH environment variable**:
    - Export PATH:
      ```bash
      export PATH='/opt/apache-maven-3.8.4':'/opt/apache-maven-3.8.4/bin':$PATH
      ```
    - Append to bashrc file:
      ```bash
      vi ~/.bashrc
      ```
    - Verify `mvn` command executes without explicit path to executable command:
      ```bash
      mvn --version
      ```

Four custom AMIs should be created as shown below with commands above.


![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/AMI's.PNG)

# VPC (Network Setup)

1. **Build VPC network (192.168.0.0/16) for Bastion Host deployment**:
   Create Bastion VPC with this basic configuration
   
   

3. **Build VPC network (172.32.0.0/16) for deploying Highly Available and Auto Scalable application servers**:
   
4. **Create NAT Gateway in Public Subnet and update Private Subnet associated Route Table**:
   - Route the default traffic to NAT for outbound internet connection.
  
5. **Create Transit Gateway**:
   - Associate both VPCs to the Transit Gateway for private communication.

6. **Create Internet Gateway for each VPC**:
   - Update Public Subnet associated Route Table accordingly to route the default traffic to IGW for inbound/outbound internet connection.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/internet_gateways.PNG)

## Bastion

1. **Deploy Bastion Host in the Public Subnet with EIP associated**.

2. **Create a Security Group allowing port 22 from public internet**.

### AWS INFRASTRUCTURE SETUP SOLUTION

Bastion and Prod VPC configuration is shown below with DNS hostnames enabled.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/bastion_vpc.PNG)


App_VPC configuration along with CIDR range is shown below
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc.PNG)

Create and attach IGW to both VPCs
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/internet_gateways.PNG)


### VPC and Route Tables Configuration

VPC and Route tables should be created as follows:

- **Bastion VPC**:
1 public subnet is created.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/bastion_vpc_subnet.PNG)

Bastion VPC needs only one route table which includes one public route to IGW, transit gateway, and local route.
  ![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/bastion_vpc_public_route_table.PNG)

 The public subnet in bastion_VPC is associated with Its public route table as shown below
 ![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/bastion_vpc_public_routetable_subnet.PNG)

SG rule for Bastion SG is restricted to a particular IP on port 22 as shown.

 ![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/bastion_security_group.PNG)


Bastion Host is spun up in the Bastion VPC’s public subnet.
 ![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/bastion_ec2.PNG)

- **App_VPC**:
  VPC and Route tables should be created as follows:
  - 1 public subnet and various private subnets are created.
  - 2 AZs are used for high availability for tomcat and nginx instances.
  - Corresponding AZs (public and private) are required for NLBs to balance traffic.
  
### Subnets CIDRs for both VPCs

Subnet CIDRs for both VPCs are created as described and shown in the screenshot below:
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_subnet.PNG)

***Route Tables***
  - 1 public route table for Public NLB.
  - At least 1 private route table for internal app, nginx, RDS, and NLB servers. (Note: Your solution may vary, my solution included 2 private route tables.)


See private and public route table rules.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/route_tables.PNG)

The Private route table in App_VPC should include a local route, a route to transit gw, and a route to natgw in a public subnet. For instance, natgw is required by the Tomcat application server to access artifacts from the Jfrog repository.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_Private_RT_internalOnly.PNG)

NAT Private Route-table in AppVPC

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_ptivate_natRoute_RT.PNG)

The public Route table in Prod_Vpc should include routes to IGW, transit gw, and local routes. This will be used by the public-facing NLB

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_public_route_table.PNG) 



Associate the public subnets to the public route table in prod_VPC

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_public_routetable_subnet.PNG) 

Associate all other private subnets to your private route table(s) in AppVPC.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_private_natRoute_subnet.PNG) 

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_private_RT_subnet.PNG) 

Using a Transit GW (TGW) provides a hub-like solution for connecting VPCs or on-premise networks to VPCs. 

Create the TGW as shown below. Only the TGW name is required.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/transit_gateway.PNG)

Two TGW attachments are required to connect both VPCs. Configuration of only Bastion TGW attachment is shown below. A similar configuration should be created for the Prod_VPC TGW attachment

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/transit_gateway_attachment.PNG)



Security groups (SG) for Bastion, Tomcat, nginx, and MySQL RDS instances are shown below. Note that a security group can’t be associated with a NLB, unlike an application load balancer.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_security_group.PNG)


RDS MySQL SG only accepts traffic from Tomcat Application SG on port 3306

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_mysql_security_group.PNG)



Nginx SG port 80 inbound rule can be further restricted to VPC CIDR instead of accepting from anywhere as shown below

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_nginx_security_group.PNG)

Tomcat server SG port 8080 rule can also be restricted to local VPC CIDR

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_tomcat_security_group.PNG)

One S3 bucket was created for pulling the nginx config and tomcat log rotation script (referenced in launch template userdata).

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_S3_bucket.PNG)


# Maven (Build)

1. **Create EC2 instance using Maven Golden AMI:**
   - Launch a Maven instance in `prodVPC`, a private subnet, with a custom Maven AMI.

2. **Clone GitHub repository to VSCode and update the `pom.xml` with Sonar and JFROG deployment details:**
   ```bash
   git clone remote_url && cd java-login-app
   git branch feature
   git checkout feature
     - Afterwards, update the `pom.xml` with the organization name and Sonar host URL.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/Sonar_pom_config.PNG)



   - *For JFrog integration:*
     - Create a repository on JFrog.
     - Use the 'Quick Setup' option to generate deployment configuration.
     - Click 'set me up' for your 'local' type repo (e.g., 'assignment-libs-release-local').
     - Click the "Deploy" tab on JFrog Web UI. This generates configuration to use at Maven to upload generated artifacts to the JFrog local repository.
     - Afterwards, update the `pom.xml` file with the generated `distributionManagement` config.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/Jfrog_pom_config.PNG)


3. **Add the `settings.xml` file to the root folder of the repository with the JFROG credentials and JFROG repo to resolve the dependencies:**
   - To generate `settings.xml`, use the 'Quick Setup' option in JFrog.
   - Select the ‘default-maven’ repo for downloading dependencies.
   - Click 'configure' using the ‘default-maven’ repo.
   - A settings configuration for Maven to connect to JFrog and download dependencies is auto-generated.
   - Place configuration in the `/root/.m2/settings.xml` file on the Maven instance. The `settings.xml` file should include credentials and references to the `default-maven-virtual` JFrog repo.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/jfrog_settiing_config.PNG)


4. **Update the `application.properties` file with the JDBC connection string to authenticate with MySQL:**
   - Update the `src/main/resources/application.properties` file with the RDS endpoint name and connection credentials.
   - 
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/application_propertise.PNG)


5. **Push the code changes to the feature branch of the GitHub repository:**
   ```bash
   git add. && git commit -m "All changes with pom and properties file"
   git push origin feature
6. **Raise Pull Request to approve the PR and Merge the changes to the Master branch:**

7. **Login to the EC2 instance and clone the Github repository:**
   ```bash
   git clone remote_repo_url && cd java-login-app

8. **Build the source code using Maven arguments (`-s settings.xml`):**
   ```bash
   mvn -s ~/.m2/settings.xml deploy
9. **Integrate Maven build with Sonar Cloud and generate an analysis dashboard with the default Quality Gate profile:**
   - As stated in step 2, execute the remaining instructions on the Maven instance.
   - Export the environment variable and run `mvn verify` command:
     ```bash
     export SONAR_TOKEN=xxxxxxxxx
     mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=assignment5
     ```
   - Analysis dashboard with a quality gate.
  
   ![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/SonarCloud.PNG)

# Database (RDS)

1. **Deploy Multi-AZ MySQL RDS instance into private subnets:**
   - This was implemented as a standalone with a Free tier for cost savings.
   - Note VPC, security group, and subnets selected.
   The endpoint name was auto-generated.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_RDB.PNG)

2. Create a Security Group allowing port 3306 from App instances and Bastion Host.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_vpc_mysql_security_group.PNG)

# Tomcat (Backend)

1. **Create private-facing Network Load Balancer and Target Group:**
   - The internal NLB listens on port 8080 and forwards to the App Target group.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_tomcat_load_balancer_listner.PNG)

App Target group is set to listen on port 8080 since Tomcat listens on the same port by default

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_target_groups.PNG)


2. **Create Launch Configuration with the following configuration:**
   1. Tomcat Golden AMI
   2. User Data to deploy .war artifact from JFROG into webapps folder.
   3. Security Group allowing Port 22 from Bastion Host and Port 8080 from private NLB.

   - When creating the launch config/template, specify the Tomcat golden AMI, keypair, App-SG, and user data, which downloads the JFROG artifact and specifies a cronjob script to rotate log files to S3.
   - Launch Template must have an IAM role for S3, SSM, and for CloudWatch agent to push custom metrics.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/LT.PNG)

To push metrics, access S3, and use session manager, the following AWS-managed policies attached to an IAM role were used in both Launch templates for Nginx and Tomcat
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/I_AM_role_policy.PNG)

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/iam_roles.PNG)

3. **Create Auto Scaling Group:**
   - The ASG is configured as shown below
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_auto_scaling_group.PNG)

# Nginx (Frontend)

1. **Create public-facing Network Load Balancer and Target Group:**
   - Nginx target group is set to port 80 since Nginx listens on port 80.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_nginx_load_balancer_lisner.PNG)



2. **Create Launch Configuration with the following configuration:**
   1. Nginx Golden AMI
   2. User Data to update `proxy_pass` rules in `nginx.conf` file and reload the Nginx service.
   3. Security Group allowing Port 22 from Bastion Host and Port 80 from Public NLB.

   - The userdata in the Nginx launch template below downloads a modified default Nginx config file from S3. An alternative solution can edit the nginx config file in place using the `sed` command. I think the former is simpler.

![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_nginx_launch_tmp.PNG)


Nginx.conf file is updated in the server block with the following directive

 3. **Create Auto Scaling Group:**  
 
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/app_auto_scaling_group.PNG)



# Application Deployment

1. **Artifact deployment taken care of by User Data script during Application tier EC2 instance launch process.**

2. **Login to MySQL database from Application Server using MySQL CLI client and create database and table schema to store the user login data:**
   - Login to tomcat server, install MySQL client, and configure DB schema:
     ```bash
     # yum install mysql -y
     # mysql -u admin -p -h database-1.cqsqkkkpnzxm.us-east-1.rds.amazonaws.com
     Enter password:
     MySQL [(none)]> CREATE DATABASE UserDB;
     MySQL [(none)]> use UserDB;
     Database changed
     MySQL [UserDB]> CREATE TABLE Employee ( id int unsigned auto_increment not null, first_name varchar(250), last_name varchar(250), email varchar(250), username varchar(250), password varchar(250), regdate timestamp, primary key (id) );
     Query OK, 0 rows affected (0.09 sec)
     ```
   

# Post-Deployment

1. **Configure Cronjob to push the Tomcat Application log data to the S3 bucket and also rotate the log data to remove the log data on the server after the data is pushed to the S3 Bucket:**
   - The Tomcat Launch template was set with a daily cronjob to execute the following script to rotate log data. An alternative could be to define a tomcat config In '/etc/logrotate.d/tomcat' and use logrotate to rotate log files.
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/CroneJob.PNG)

2. **Configure CloudWatch alarms to send E-Mail notifications when database connections are more than 100 threshold.**


# Validation

1. **Verify you, as an administrator, can login to EC2 instances from the session manager & Bastion Host:**
   - SSH forwarding was used to access Nginx and App instances from the bastion host.
   - Start the SSH agent and add the key you want to forward to the agent:
     ```bash
     $ eval "$(ssh-agent)"
     Agent pid 83448
     $ ssh-add valaxy5.pem && ssh -A ec2-user@100.24.42.154
     ```
   - Confirmed access to Tomcat instance from the bastion.

Session Manager access for Nginx aided with the IAM role defined in the launch template
Session Manager access for Tomcat aided with the IAM role defined in the launch template



2. **Verify if you, as an end user, can access the application from the public internet browser:**
  

The following user “UserDB” successfully registered via the website
![Graphical user interface, text, and application Description automatically
generated](https://github.com/Hashirahmad996/java-application-deploy-on-aws/blob/master/project%20guide/images/DB_check.PNG)


