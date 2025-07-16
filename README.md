# 3tier-webapp-deployment
Scalable and Highly Available Web Application Deployment using 3-Tier Architecture on AWS

## Introduction

Deploying web applications with high availability, scalability, and security is a critical requirement in modern cloud infrastructure. This project demonstrates the implementation of a **3-Tier Architecture** on **Amazon Web Services (AWS)** to host a web application in a production-ready environment.

The 3-Tier model is a widely adopted architectural pattern that separates application components into three distinct layers:

- **Presentation Tier** – the frontend, handling user interaction.
- **Application Tier** – the business logic or backend processing.
- **Database Tier** – the data layer that stores and manages application data.

This architecture is deployed across multiple **Availability Zones (AZs)** in AWS to ensure high availability, using components such as:

- Elastic Load Balancers (ELBs)
- Auto Scaling Groups (ASGs)
- Amazon RDS (Relational Database Service)
- Secure networking using VPCs

The goal is to simulate real-world cloud infrastructure for hosting modern, scalable, and fault-tolerant applications.

---

## ❓ Why 3-Tier Architecture

The **3-tier architecture** offers modularity, scalability, and security by separating the application into three distinct functional layers:

1. **Presentation Tier (Frontend)**  
   The user interface layer—typically a browser or client app. Hosting this separately enables independent scaling based on traffic.

2. **Application Tier (Logic Layer)**  
   Handles backend processing, session management, and business logic. This isolation improves maintainability and performance.

3. **Database Tier (Data Layer)**  
   Manages storage, retrieval, and security of application data. Separation of this layer enhances integrity and protection.

This separation ensures flexibility, easier management, and improved fault isolation across the application stack.

---

## Task 1: Creating a VPC and Subnets

1. Go to **VPC Dashboard** → click **Create VPC**.
2. Select **VPC and more**, name the VPC, use CIDR block `10.0.0.0/16`.
3. Select **2 AZs**, create **2 public** and **4 private subnets**.
4. Customize subnet IP ranges, configure **NAT Gateway** in one AZ, then click **Create VPC**.
5. Enable **Auto-assign public IPv4** for all subnets under subnet settings.
6. Confirm route tables are associated correctly.

---

## Task 2: Creating Web Server Tier (Frontend)

1. Go to **EC2 Dashboard** → click **Launch Instance**.
2. Select **Amazon Linux AMI**, type `t2.micro`, enable public IP, and attach to public subnet.
3. Create a **security group** allowing `SSH`, `HTTP`, and `HTTPS` from anywhere (for demo).
4. In **Advanced Details**, use the **user-data script** to install Apache web server:
   ```bash
   #!/bin/bash
   yum update -y
   yum install -y httpd
   systemctl start httpd
   systemctl enable httpd
   echo "Welcome to the Web Tier - $(hostname)" > /var/www/html/index.html
5. Launch the instance, copy its public IP, and verify the Apache page.
  -Create Launch Template & Auto Scaling Group
 1. Go to Launch Templates → create a new one using above config.
 2. Enable public IP and use same Apache script.
 3. Create Auto Scaling Group (ASG):
  -Attach to Launch Template
  -Use public subnets and internet-facing ALB
  -Set group size: min 2, max 3, desired 2
  -Configure CPU tracking (target 50%)
  -Enable CloudWatch metrics
 4. Verify load balancer DNS URL loads the web page.

## Task 3: Creating Application Tier (Middleware)

1. Create a new launch template for app tier EC2:
 -Use same AMI
 -Type: t2.micro
 -New security group with:
 -SSH (from anywhere)
   -Custom TCP (from Web Tier SG)
   -ALL ICMP (from anywhere)
   -Use same user-data script
2. Create Auto Scaling Group:
 -Attach new Launch Template
 -Use private subnets
 -Use internal-facing ALB
 -Set same scaling policy as web tier
3.Validate:
 -You cannot SSH directly from the internet
 -You can ping app-tier EC2 from web-tier EC2

## Task 4: Creating Database Tier (RDS)

1. Go to RDS Console → create Subnet Group
 -Include private subnets only
2.Create a new MySQL DB instance:
 -Select Free-tier eligible options
 -Choose VPC and created subnet group
 -Create a new security group
3.Edit RDS Security Group:
 -Remove rule allowing public IP
 -Add MySQL/Aurora (3306) access from Application Tier SG
4.In Route Tables, ensure private subnets are associated with correct routes.

## Task 5: Testing Connectivity

SSH into Database (via Web → App)
1. Add key to SSH agent:
  -ssh-add your-key.pem
2. SSH into Web Tier:
  -ssh -A ec2-user@<public-ip-of-web-instance>
3. From Web Tier → SSH into App Tier:
 -ssh -A ec2-user@<private-ip-of-app-instance>
4. Install MySQL client:
 -sudo dnf install mariadb105-server
5. Connect to RDS:
 -mysql -h <rds-endpoint> -u <db-username> -p
6.Successfully access the DB from App Tier

## Conclusion
By implementing a 3-tier architecture on AWS, we built a system that is:
-Scalable via Auto Scaling and ALB
-Highly Available using Multi-AZ deployment
-Secure with layered subnets, NACLs, and IAM
-Maintainable with separation of concerns across tiers
This architecture mirrors real-world best practices and serves as a solid foundation for deploying cloud-native, production-grade web applications.

 

