# ec2-open-cart-deployment
Deploy an **OpenCart** e-commerce platform on a **Linux EC2 instance** using the **AWS Management Console (GUI)**. Configure **secure storage** with EBS encryption, assign an **Elastic IP**, and optionally use **IAM roles** for S3 backup access.

# 🛒 OpenCart Deployment on AWS EC2

## 🎯 Goal

Deploy an **OpenCart** e-commerce platform on a **Linux EC2 instance** using the **AWS Management Console (GUI)**. Configure **secure storage** with EBS encryption, assign an **Elastic IP**, and optionally use **IAM roles** for S3 backup access.

---

## ✅ Task 1: Create EC2 Instance

### 🔹 Step 1.1: Launch Instance

1. Go to: `EC2 > Instances > Launch Instances`
2. Configure instance:
   - **Name**: `OpenCart-Server`
   - **AMI**: Amazon Linux 2 or Ubuntu 20.04
   - **Instance Type**: `t2.micro` (Free Tier eligible)
   - **Key Pair**: Choose existing or create a new one
   - **Network Settings**:
     - VPC: Default
     - Auto-assign Public IP: Enabled

### 🔹 Step 1.2: Configure Security Group

- Create a new Security Group named `OpenCartSG`
- Add Inbound Rules:
  - **HTTP (port 80)** – Source: Anywhere (0.0.0.0/0)
  - **SSH (port 22)** – Source: Your IP

### 🔹 Step 1.3: Add User Data Script

Paste the following into **Advanced Details > User data**:

```bash
#!/bin/bash
yum update -y
yum install -y httpd php php-mysqlnd wget unzip

systemctl start httpd
systemctl enable httpd

cd /var/www/html
wget https://github.com/opencart/opencart/releases/download/3.0.3.8/opencart-3.0.3.8.zip
unzip opencart-3.0.3.8.zip
cp -r upload/* .
rm -rf upload opencart-3.0.3.8.zip

chown -R apache:apache /var/www/html
chmod -R 755 /var/www/html
