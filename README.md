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
```

# 🛒 OpenCart Deployment – Part 2

---

## ✅ Task 2: Attach & Encrypt EBS Volume

### 🔹 Step 2.1: Create Volume

Go to: `EC2 > Volumes > Create Volume`

**Configure:**

- **Size**: 20 GiB  
- **Type**: gp3  
- **Encryption**: ✅ Enabled  
- **Availability Zone**: Match your EC2 instance (e.g., `us-east-1a`)  
- **Tag**: `Name=OpenCartData`

---

### 🔹 Step 2.2: Attach Volume

1. Select the volume → `Actions > Attach Volume`
2. Choose your instance
3. Device name: `/dev/xvdbc`

---

### 🔹 Step 2.3: Format and Mount Volume

SSH into your EC2 instance and run:

```bash
sudo mkfs -t ext4 /dev/xvdbc
sudo mkdir /mnt/data
sudo mount /dev/xvdbc /mnt/data
```
## ✅ Task 3: Assign Elastic IP

### 🔹 Step 3.1: Allocate Elastic IP

1. Go to: `EC2 > Elastic IPs`
2. Click **Allocate Elastic IP address**
3. Confirm and proceed

---

### 🔹 Step 3.2: Associate Elastic IP

1. Select the allocated Elastic IP
2. Click **Actions > Associate Elastic IP**
3. Choose your EC2 instance

➡️ Your EC2 instance now has a **static public IP** that persists across reboots.

## ✅ Task 4 (Optional): IAM Role for S3 Backups

### 🔹 Step 4.1: Create IAM Role

1. Go to: `IAM > Roles > Create Role`
2. **Trusted entity**: AWS Service
3. **Use Case**: EC2
4. **Permissions**: Attach `AmazonS3FullAccess` or use a custom policy:

```json
{
  "Effect": "Allow",
  "Action": ["s3:PutObject", "s3:GetObject", "s3:ListBucket"],
  "Resource": ["arn:aws:s3:::your-bucket-name/*"]
}
```

## 🔹 Step 4.2: Attach IAM Role to EC2

1. Go to: `EC2 > Instances`
2. Select your instance → `Actions > Security > Modify IAM Role`
3. Attach: `OpenCartS3BackupRole`

---

## 🧪 Additional Commands Used (Manual Actions)

```bash
# Initial formatting and mounting attempts
sudo mkfs -t ext4 /dev/xvdbf
sudo mkfs -t ext4 /dev/xvdbc
df -h
lsblk
sudo mkdir /mnt/data
sudo mount /dev/xvdbc /mnt/data
```

# Persist the mount after reboot
``` bash
echo "/dev/xvdbc /mnt/data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
```

# PHP package update and install
``` bash
sudo yum remove php*
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-mysqlnd php-fpm php-opcache php-gd php-xml php-mbstring -y
sudo systemctl restart httpd
php -v
``` 

![Image](https://github.com/user-attachments/assets/c0979ba5-d24c-4caa-bd89-1855cf3a849e)
