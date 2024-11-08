
# Highly Available Three-Tier Web Application on AWS

## Project Overview

This project demonstrates the deployment of a **highly available**, **fault-tolerant**, and **scalable three-tier web application** using Amazon Web Services (AWS). The three-tier architecture consists of a **web tier**, **application tier**, and **database tier**, designed to ensure reliability, scalability, and security for modern web applications.

### Key Components
- **Web Tier**: Manages incoming user traffic through Elastic Load Balancer (ELB) and serves static content.
- **Application Tier**: Handles business logic and dynamic content, deployed on EC2 instances with Auto Scaling.
- **Database Tier**: Manages data storage using Amazon RDS (Aurora/MySQL) with Multi-AZ for high availability.

---

## Features

- **High Availability**: The application is deployed across multiple availability zones (AZs) to ensure resilience and uptime.
- **Auto Scaling**: Automatically adjusts capacity to meet traffic demands in the web and application tiers.
- **Load Balancing**: Uses AWS Elastic Load Balancer (ELB) to distribute incoming traffic across healthy instances.
- **Database Failover**: Amazon RDS is used for the database layer, ensuring automatic failover and backups.
- **Monitoring & Security**: Integrated with AWS CloudWatch for monitoring and IAM roles for secure access to resources.

---

## Architecture

![](https://github.com/Kishor-Bibin/Highly-Available-Three-Tier-Web-Application-on-AWS-/blob/649c035a890dced071a27c6e18f76627699c541c/Images/Three-tier-Architecture%20Diagram.png)
### The architecture includes:
1. **Web Layer (Public Subnet)**:
   - EC2 instances running **Nginx** web servers.
   - Elastic Load Balancer (ELB) in front to handle incoming traffic.
   - Auto Scaling Group (ASG) to adjust based on traffic.
2. **Application Layer (Private Subnet)**:
   - EC2 instances for application logic.
   - Deployed in private subnets with access restricted through security groups.
3. **Database Layer (Private Subnet)**:
   - Amazon RDS with **Multi-AZ** support for database storage.
   - RDS security groups restrict access to the application layer only.

---

## How It Works

1. **User Traffic**: Users access the application through the public URL, which is routed through the **Elastic Load Balancer**.
2. **Web Tier**: The load balancer forwards requests to EC2 instances running **Nginx**, serving static content.
3. **Application Tier**: Dynamic requests are processed by the application layer, hosted on EC2 instances behind the **Auto Scaling Group** for load handling.
4. **Database Tier**: Application servers interact with **Amazon RDS** for database operations, ensuring data persistence and backup.
5. **Auto Scaling**: Based on traffic patterns, the web and application tiers automatically scale up or down to handle the load.
6. **Monitoring**: AWS **CloudWatch** is used to monitor performance metrics, and alarms are set to trigger scaling or notifications if necessary.

---

## Prerequisites

- **AWS Account** with sufficient permissions.
- **AWS CLI** and **Terraform** (optional) for automated infrastructure setup.
- Basic understanding of **AWS services**: EC2, RDS, VPC, and Auto Scaling.

---

## Setup and Deployment

To deploy this project, follow these steps:

### 1. Clone the Repository

```bash
git clone https://github.com/your-repo/three-tier-aws-app.git
cd three-tier-aws-app
```

### 2. Create Networking

1. Set up a **VPC** with subnets across multiple Availability Zones.
2. Create **Internet Gateway**, **NAT Gateway**, and configure routing tables.

### 3. Deploy EC2 Instances for Web and Application Tiers

- Launch EC2 instances with **Auto Scaling Groups** for both web and application tiers.
- Install **Nginx** on web tier and configure it to serve static content.

### 4. Create Amazon RDS

- Set up **Amazon Aurora** or **MySQL RDS** in private subnets with Multi-AZ configuration.

### 5. Configure Security

- Ensure correct **security groups** for restricting access between tiers.
- Use **IAM Roles** to provide least-privilege access to resources like S3 or CloudWatch.

### 6. Set Up Load Balancing and Auto Scaling

- Configure **Elastic Load Balancer** to distribute traffic.
- Enable **Auto Scaling** to automatically adjust instance counts based on demand.

---

## Testing

1. Once deployed, test the application by accessing the **Elastic Load Balancer's DNS name**.
2. Verify traffic is routed properly, and the application scales with increasing load.
3. Use **CloudWatch** to monitor instance health, performance metrics, and check for any scaling events.

---

## Monitoring and Logging

- **CloudWatch** monitors the health and performance of the EC2 instances, load balancer, and RDS.
- **VPC Flow Logs** capture traffic flow data for analysis and security auditing.

---

## Future Improvements

- Integrating **CI/CD pipeline** for continuous integration and deployment.
- Implementing **WAF (Web Application Firewall)** for additional security layers.
- Adding **CloudFront** for content caching and improving application performance globally.

---
