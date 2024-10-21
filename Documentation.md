
# Highly Available Three-Tier Web Application on AWS

## Overview

This project implements a highly available and scalable three-tier web application architecture on AWS. The architecture consists of a web tier, application tier, and database tier, using services like **EC2**, **Elastic Load Balancing**, **Auto Scaling**, **Amazon RDS**, and **S3**. The system is designed for fault tolerance, security, and scalability.

## Architecture Diagram

**Include a diagram here** showing the relationship between the web, application, and database layers, including security groups, load balancers, subnets, and other AWS resources.

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
[screenshot]

#### 1.2. Create Subnets
- Create **public subnets** for web servers and **private subnets** for the application and database servers in multiple Availability Zones.
  [Screenhot]

#### 1.3. Create Internet Gateway and NAT Gateway
- Create an **Internet Gateway** for public-facing web servers and attact it to the VPC created before.
  [screenshot]
- Create **NAT Gateway** in a public subnet to allow private instances to access the internet.
[Screenshot]

#### 1.4. Configure Routing Tables

In an AWS VPC, routing tables are essential to direct traffic between subnets and internet gateways (IGWs) or NAT gateways (NAT GWs).

#### 1.4.1. Route Tables for Public Subnets (with Internet Gateway)
Public subnets are subnets that have direct access to the internet via an Internet Gateway (IGW). This configuration is used when instances in the subnet need to communicate directly with external networks.

#### Create a Route Table for Public Subnet:

Go to VPC *Dashboard > Route Tables > Create Route Table*.

Name it something like Public Route Table.
Associate the public subnet with this route table.
Add Route to the Internet Gateway:

Edit the route table to add a route for internet-bound traffic.
Destination: 0.0.0.0/0 (which means all traffic).
Target: Select the Internet Gateway (IGW) associated with the VPC.
Associate the Route Table with the Public Subnet:

Go to Subnet Associations under the route table.
Choose the public subnet you wish to associate this route table with.

### 1.4.2. Route Tables for Private Subnets (with NAT Gateway)

Private subnets do not have direct access to the internet, but instances within them can access the internet via a NAT Gateway (NAT GW). This is common for instances that need to download updates or communicate outbound without being directly reachable from the internet.

#### Create a Route Table for Private Subnet:

Go to *VPC Dashboard > Route Tables > Create Route Table*.
[screenshot]

Name it something like Private Route Table.
Associate the private subnet with this route table.
Add Route to the NAT Gateway:

Edit the route table to add a route for internet-bound traffic.
Destination: 0.0.0.0/0 (for all internet traffic).

Target: Select the NAT Gateway created in the public subnet.
Associate the Route Table with the Private Subnet:

Go to Subnet Associations under the route table.
Choose the private subnet you want to associate this route table with.

### 2. Web Tier

#### 2.1. Launch EC2 Instances for Web Servers
```bash
aws ec2 run-instances --image-id <ami-id> --instance-type t2.micro --subnet-id <public-subnet-id> --key-name <key-name>
```
- Use an **Amazon Linux 2** or **Ubuntu** AMI for the EC2 instances hosting the web servers.

#### 2.2. Configure Security Groups
- Create a **security group** to allow inbound HTTP (port 80) and HTTPS (port 443) traffic.

#### 2.3. Install and Configure Nginx
```bash
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```
- **Nginx** will act as the web server to route traffic to the application tier.

### 3. Application Tier

#### 3.1. Set Up EC2 Instances for Application Layer
- Launch EC2 instances in the private subnet, using an **application-specific AMI** or base OS like Amazon Linux 2.

#### 3.2. Configure Application Tier Security
- Create a security group to allow **inbound traffic** only from the web tier’s security group.
  
#### 3.3. Application Code Deployment
- Use **S3** to store and download the application code on your EC2 instances:
```bash
aws s3 cp s3://<bucket-name>/app-code /var/www/html --recursive
```
  
#### 3.4. Load Balancer for Application Layer
- Create an **internal Application Load Balancer (ALB)** to distribute traffic across application servers.

### 4. Database Tier

#### 4.1. Create Amazon RDS Instance
- Provision an **Amazon Aurora** or **MySQL RDS** instance in the private subnets:
```bash
aws rds create-db-instance --db-instance-identifier mydb --db-instance-class db.t3.micro --engine aurora --allocated-storage 20 --master-username admin --master-user-password <password>
```
- Ensure Multi-AZ is enabled for high availability.

#### 4.2. Database Security
- Create a security group that only allows traffic from the **application tier’s security group** (MySQL port 3306).

#### 4.3. Configure Database Subnet Group
- Ensure the RDS instance spans across multiple subnets for high availability.

### 5. Auto Scaling and Load Balancing

#### 5.1. Create Auto Scaling Groups (ASG)
- Define **Auto Scaling Groups** for both web and application tiers to automatically scale based on traffic.

#### 5.2. Configure Health Checks
- Set up **health checks** in the **load balancer** to ensure traffic is only routed to healthy instances.

### 6. Security Configuration

#### 6.1. IAM Roles and Policies
- Create **IAM Roles** for your EC2 instances to allow access to **S3** and **CloudWatch**.
  
#### 6.2. Monitoring and Logging
- Use **CloudWatch** to monitor the health of your infrastructure and application.
- Enable **VPC Flow Logs** for network security monitoring.

### 7. Verifying the Setup

1. Access your web application via the **Elastic Load Balancer’s DNS name**.
2. Ensure the application can access the database and properly serves requests from the web tier.
3. Verify auto-scaling by simulating traffic.

### 8. Clean Up Resources

To avoid unnecessary costs, delete the resources after testing:
```bash
aws ec2 terminate-instances --instance-ids <instance-id>
aws rds delete-db-instance --db-instance-identifier mydb --skip-final-snapshot
aws s3 rb s3://<bucket-name> --force
```

## Troubleshooting

- **EC2 Connectivity Issues**: Check your **security group** rules and ensure you have the right **key pair**.
- **Database Connection Failures**: Ensure proper security group rules and subnet configuration for **RDS**.
- **Scaling Issues**: Verify the **Auto Scaling policies** and instance health checks.

## License

This project is licensed under the MIT License.

---

This README is designed to guide users through the process of deploying a highly available three-tier application on AWS.
