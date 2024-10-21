
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

#### 1.4.2. Route Tables for Private Subnets (with NAT Gateway)

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

#### 1.5. Configure Security Groups
Security groups act as virtual firewalls that control inbound and outbound traffic to and from your instances. When configuring security groups for your VPC, you’ll need separate rules for instances in the public subnets and private subnets to ensure appropriate security. Below are the steps to create security groups for both public and private subnets:

**1.5.1 Create a Security Group for ELB**
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

**1.5.2 Create a security group for Web-tier**
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

**1.5.3 Create a security group for Internal Load Balancer**
-

Name it something like **Internal-lb-sg** and associate it with the VPC

Inbound rules for Internal-lb-SG : 
``` bash
Type : HTTP
Protocol: Custom TCP
Port Range: 80
Source: Web-tier-SG
```
**1.5.4 Create a security group for App-Tier**
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
Type ; Custom TCP
Protocol: TCP
Port Range: 4000
Source: My IP
```
**1.5.5 Create a security group for Database Tier**
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


