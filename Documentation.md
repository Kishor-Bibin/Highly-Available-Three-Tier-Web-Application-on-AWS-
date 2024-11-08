
# Highly Available Three-Tier Web Application on AWS

## Overview

This project implements a highly available and scalable three-tier web application architecture on AWS. The architecture consists of a web tier, application tier, and database tier, using services like **EC2**, **Elastic Load Balancing**, **Auto Scaling**, **Amazon RDS**, and **S3**. The system is designed for fault tolerance, security, and scalability.

## Architecture Diagram

![](https://github.com/Kishor-Bibin/Highly-Available-Three-Tier-Web-Application-on-AWS-/blob/695e40e15298ff396d6abad74e0b09599d88f4a9/Images/Three-tier-Architecture%20Diagram.png)

## Prerequisites

Before you begin, ensure you have the following:
- **AWS Account** with required permissions.
- **AWS CLI** installed and configured.
- Basic knowledge of **AWS services** (EC2, RDS, S3, VPC, etc.).
- Basic Knowledge of **Git**

## Steps to Deploy

### Setup and Deployment

To deploy this project, follow these steps:

### 0. Clone the Repository
Open Git Bash

```bash
git clone https://github.com/aws-samples/aws-three-tier-web-architecture-workshop.git
cd aws-three-tier-web-architecture-workshop
```
This repo contains all the nesessary app code 

### 1. Setting Up Networking

#### 1.1. Create a VPC
- Create a **VPC** to isolate resources with a CIDR block of 10.0.0.0/16.
![](https://github.com/Kishor-Bibin/Highly-Available-Three-Tier-Web-Application-on-AWS-/blob/4d658c2d7afde140e05783a63ba72d843ff741d9/Images/vpc-create.png)


#### 1.2. Create Subnets
- Create **public subnets** for web servers and **private subnets** for the application and database servers in multiple Availability Zones.
  ![](https://github.com/Kishor-Bibin/Highly-Available-Three-Tier-Web-Application-on-AWS-/blob/4d658c2d7afde140e05783a63ba72d843ff741d9/Images/Subnet-create.png)

#### 1.3. Create Internet Gateway and NAT Gateway
- Create an **Internet Gateway** for public-facing web servers and attact it to the VPC created before.
  ![](https://github.com/Kishor-Bibin/Highly-Available-Three-Tier-Web-Application-on-AWS-/blob/4d658c2d7afde140e05783a63ba72d843ff741d9/Images/IGW-create.png)
- Create **NAT Gateway** in a public subnet to allow private instances to access the internet.
![](https://github.com/Kishor-Bibin/Highly-Available-Three-Tier-Web-Application-on-AWS-/blob/4d658c2d7afde140e05783a63ba72d843ff741d9/Images/NAT-gateway%20create.png)  


#### 1.4. Configure Routing Tables

In an AWS VPC, routing tables are essential to direct traffic between subnets and internet gateways (IGWs) or NAT gateways (NAT GWs).



#### 1.4.1 Route Tables for Public Subnets (with Internet Gateway)

Public subnets allow instances to access the internet via an Internet Gateway (IGW). This setup is used when instances in the subnet need to communicate directly with external networks.

#### Steps to Create a Route Table for a Public Subnet

1. **Go to the VPC Dashboard**  
   - Navigate to **VPC Dashboard > Route Tables > Create Route Table**.
   
2. **Name the Route Table**  
   - Give it a descriptive name, like `Public Route Table`.

3. **Associate the Public Subnet**  
   - Associate the public subnet with this route table.

4. **Add a Route for Internet-Bound Traffic**  
   - Edit the route table and add a route for traffic destined for the internet:
     - **Destination**: `0.0.0.0/0` (this represents all traffic).
     - **Target**: Select the Internet Gateway (IGW) associated with your VPC.

5. **Associate the Route Table with the Public Subnet**  
   - Go to the **Subnet Associations** tab under the route table.
   - Select the public subnet to associate it with this route table.

---

#### 1.4.2 Route Tables for Private Subnets (with NAT Gateway)

Private subnets don’t have direct internet access, but instances can access the internet via a NAT Gateway (NAT GW). This is useful when instances need to communicate outbound without being accessible from the internet.

#### Steps to Create a Route Table for a Private Subnet

1. **Go to the VPC Dashboard**  
   - Navigate to **VPC Dashboard > Route Tables > Create Route Table**.

2. **Name the Route Table**  
   - Name it something descriptive, like `Private Route Table`.

3. **Associate the Private Subnet**  
   - Associate the private subnet with this route table.

4. **Add a Route for Internet-Bound Traffic via NAT Gateway**  
   - Edit the route table and add a route for internet traffic:
     - **Destination**: `0.0.0.0/0` (for all internet traffic).
     - **Target**: Select the NAT Gateway created in the public subnet.

5. **Associate the Route Table with the Private Subnet**  
   - Go to the **Subnet Associations** tab under the route table.
   - Select the private subnet to associate it with this route table.




### 1.5. Configure Security Groups
Security groups act as virtual firewalls that control inbound and outbound traffic to and from your instances. When configuring security groups for your VPC, you’ll need separate rules for instances in the public subnets and private subnets to ensure appropriate security. Below are the steps to create security groups for both public and private subnets:

#### 1.5.1 Create a Security Group for ELB
---

Go to *VPC Dashboard > Security Groups > Create Security Group*.

Name it something like **InternetFacing-SG** and associate it with the VPC.

Inbound Rules for External Load Balancer Security Group:
``` bash
Inbound Rule 1 
Allow HTTP (Port 80):
Type: HTTP
Protocol: TCP
Port Range: 80
Source: 0.0.0.0/0 (allows access from any IP address).
```

#### 1.5.2 Create a security group for Web-tier
---

Name it something like web-tier-SG and associate it with the VPC

Inbound rules for Web-Tier-SG:
``` bash
Allow HTTP (port 80)
Type : HTTP
Protocol: TCP
Port Range: 80
Source: InternetFacing-SG
```

#### 1.5.3 Create a security group for Internal Load Balancer
---

Name it something like **Internal-lb-sg** and associate it with the VPC

Inbound rules for Internal-lb-SG : 
``` bash
Type : HTTP
Protocol: Custom TCP
Port Range: 80
Source: Web-tier-SG
```
#### 1.5.4 Create a security group for App-Tier
-

Name it something like **app-tier-sg** and associate it with the VPC

Inbound rules for Internal-lb-sg : 
``` bash
Type : Custom TCP
Protocol: TCP
Port Range: 4000
Source: Internal-lb-SG
```
Inbound rules 2 for Internal-lb-sg : 
``` bash
Type : Custom TCP
Protocol: TCP
Port Range: 4000
Source: My IP
```
#### 1.5.5 Create a security group for Database Tier
---

Name it something like **DB-SG** and associate it with the VPC

Inbound rules for Internal-lb-sg : 
``` bash
Type : Custom TCP
Protocol: TCP
Port Range: 3306
Source: app-tier-sg
```



## 2. Web Tier

The web tier will contain the EC2 instances that act as web servers, responsible for serving the frontend of the application. Here’s how to set up and configure the web tier:

### 2.1 Launch EC2 Instance for Web Server

1. **Go to the EC2 Dashboard**:
   - Navigate to **EC2 > Instances > Launch Instances**.

2. **Choose AMI**:
   - Select an Amazon Linux 2 AMI (free tier eligible).

3. **Choose Instance Type**:
   - Select **t2.micro** (or another instance type as per project requirements).

4. **Configure Network Settings**:
   - Choose the VPC created earlier.
   - Select a **Public Subnet** for the web server instance.
   - **Enable Auto-Assign Public IP** to ensure the instance can communicate over the internet.

5. **Attach Security Group**:
   - Select the **web-tier-SG** created in Step 1.5.2.

6. **Launch the Instance**:
   - Review settings and launch the instance with a key pair for SSH access.

### 2.2 Install and Configure Nginx

1. **Connect to the EC2 Instance**:
   - Use the key pair and connect via SSH:
     ```bash
     ssh -i "your-key-pair.pem" ec2-user@public-IP-address
     ```

2. **Install Nginx**:
   - Update packages and install Nginx:
     ```bash
     sudo yum update -y
     sudo amazon-linux-extras install nginx1 -y
     ```

3. **Configure Nginx**:
   - Open the Nginx configuration file and point it to the **Internal Load Balancer** for backend routing:
     ```bash
     sudo nano /etc/nginx/nginx.conf
     ```
   - Update the server block to route traffic to the internal load balancer. Replace the default configuration with the load balancer’s DNS.

4. **Start Nginx**:
   - Start and enable Nginx to run on instance boot:
     ```bash
     sudo systemctl start nginx
     sudo systemctl enable nginx
     ```

5. **Verify Configuration**:
   - Open the web server’s public IP in a browser to ensure it serves content correctly.

6. **Create AMI for Scaling**:
   - After configuring, create an AMI from this instance for launching additional instances in the web tier.

### 2.3 Configure Target Group and External Load Balancer

1. **Create a Target Group**:
   - Go to **EC2 > Load Balancers > Target Groups** and create a target group for the web servers.
   - Choose **Instances** as the target type and associate it with the **public subnets**.

2. **Create an External Application Load Balancer**:
   - Go to **Load Balancers > Create Load Balancer** and choose **Application Load Balancer**.
   - Select **Internet-facing** and associate it with the public subnets.
   - Configure listeners and target group to route traffic from the load balancer to the web tier target group.

---

## 3. App Tier

The application tier is responsible for hosting the backend services. This tier includes an EC2 instance configured to handle server-side code and communicate with the database tier.

### 3.1 Launch EC2 Instance for Application Server

1. **Go to the EC2 Dashboard**:
   - Navigate to **EC2 > Instances > Launch Instances**.

2. **Choose AMI**:
   - Select the AMI created in Step 2.2.

3. **Choose Instance Type**:
   - Select **t2.micro** (or as per project requirements).

4. **Configure Network Settings**:
   - Choose the VPC created earlier.
   - Select a **Private Subnet** for the app server instance.

5. **Attach Security Group**:
   - Select the **app-tier-SG** created in Step 1.5.4.

6. **Launch the Instance**:
   - Review settings and launch with a key pair.

### 3.2 Install Application Dependencies

1. **Connect to the EC2 Instance**:
   - Use SSH to connect to the instance.

2. **Install Node.js and Other Packages**:
   - Install NVM, Node.js, and other necessary dependencies for the application.
   - Install **pm2** to manage the Node.js application.

3. **Configure Database Connection**:
   - Modify the application code to connect to the database using the Aurora database endpoint.

4. **Test the Application**:
   - Run the backend code to verify connectivity to the database and check for errors.

### 3.3 Create Internal Load Balancer for App Tier

1. **Create a Target Group**:
   - Go to **Target Groups** and create a target group for the app tier.
   - Select **Instances** as the target type and associate it with the private subnets.

2. **Create an Internal Load Balancer**:
   - Go to **Load Balancers** and create an **Application Load Balancer**.
   - Choose **Internal** and associate it with the private subnets.
   - Configure listeners and target group for routing traffic from the web tier to the app tier.

3. **Create Launch Template and Auto Scaling Group**:
   - Use the AMI created earlier for the app tier to create a launch template.
   - Set up an Auto Scaling Group to handle scaling based on demand.

---

## Final Steps and Testing

After deploying all the tiers, verify the following:

1. **Access the Web Application**:
   - Use the DNS name of the external load balancer to access the application.

2. **Test Database Connectivity**:
   - Add and retrieve records to ensure database connectivity.

3. **Monitor Auto Scaling**:
   - Confirm that auto-scaling works by increasing or decreasing the traffic to see how instances adjust.

4. **Clean Up**:
   - After testing, delete all resources to avoid unnecessary costs.

---




