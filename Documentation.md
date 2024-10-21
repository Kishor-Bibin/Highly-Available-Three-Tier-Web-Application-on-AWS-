
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

### Steps to Create a Route Table for a Public Subnet

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

### Steps to Create a Route Table for a Private Subnet

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
-

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

### 1.5.2 Create a security group for Web-tier
-

Name it something like web-tier-SG and associate it with the VPC

Inbound rules for Web-Tier-SG:
``` bash
Allow HTTP (port 80)
Type : HTTP
Protocol: TCP
Port Range: 80
Source: InternetFacing-SG
```

### 1.5.3 Create a security group for Internal Load Balancer
-

Name it something like **Internal-lb-sg** and associate it with the VPC

Inbound rules for Internal-lb-SG : 
``` bash
Type : HTTP
Protocol: Custom TCP
Port Range: 80
Source: Web-tier-SG
```
### 1.5.4 Create a security group for App-Tier
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
### 1.5.5 Create a security group for Database Tier
-

Name it something like **DB-SG** and associate it with the VPC

Inbound rules for Internal-lb-sg : 
``` bash
Type : Custom TCP
Protocol: TCP
Port Range: 3306
Source: app-tier-sg
```

## 2. Web Tier


