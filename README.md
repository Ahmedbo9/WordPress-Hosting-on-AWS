
# WordPress Hosting on AWS

This project involves hosting a WordPress website on AWS, utilizing various AWS resources to ensure scalability, reliability, and security. Below are the steps and scripts used to deploy the web application on an EC2 instance within a robust AWS infrastructure.

## Table of Contents

- [Architecture](#architecture)
- [AWS Resources Used](#aws-resources-used)
- [Installation Script](#installation-script)
- [Auto Scaling Group Launch Template Script](#auto-scaling-group-launch-template-script)

## Architecture

The architecture of this project includes the following components:
- A Virtual Private Cloud (VPC) with both public and private subnets across two availability zones.
- An Internet Gateway for connectivity between VPC instances and the internet.
- Security Groups as a network firewall mechanism.
- Public Subnets for infrastructure components like the NAT Gateway and Application Load Balancer.
- Private Subnets for web servers (EC2 instances) to enhance security.
- An Application Load Balancer to distribute web traffic to an Auto Scaling Group of EC2 instances across multiple availability zones.
- An Auto Scaling Group to manage EC2 instances for availability, scalability, fault tolerance, and elasticity.
- Simple Notification Service (SNS) for alerting activities within the Auto Scaling Group.
- A domain name registered and a DNS record set up using Route 53.
- Amazon Elastic File System (EFS) for a shared file system.
- Amazon RDS for database storage.
- SSL/TLS certificates managed by AWS Certificate Manager for secure communications.

## AWS Resources Used

1. **Virtual Private Cloud (VPC)**: Configured with public and private subnets across two availability zones.
2. **Internet Gateway**: Facilitates connectivity between VPC instances and the internet.
3. **Security Groups**: Network firewall mechanisms.
4. **Availability Zones**: Leveraged for enhanced system reliability and fault tolerance.
5. **Public Subnets**: Used for the NAT Gateway and Application Load Balancer.
6. **EC2 Instance Connect Endpoint**: For secure connections to assets within both public and private subnets.
7. **Private Subnets**: Web servers positioned within these subnets for enhanced security.
8. **NAT Gateway**: Enables instances in private subnets to access the internet.
9. **EC2 Instances**: Hosted the WordPress website.
10. **Application Load Balancer**: Evenly distributes web traffic.
11. **Auto Scaling Group**: Automatically manages EC2 instances.
12. **GitHub**: Stores web files for version control and collaboration.
13. **Certificate Manager**: Secures application communications.
14. **Simple Notification Service (SNS)**: Alerts about activities within the Auto Scaling Group.
15. **Route 53**: Registers the domain name and sets up DNS records.
16. **Amazon EFS**: Used for shared file systems.
17. **Amazon RDS**: Used for database storage.

## Installation Script

```bash
#!/bin/bash

# Switch to root user
sudo su

# Update software packages
sudo yum update -y

# Create HTML directory
sudo mkdir -p /var/www/html

# Set environment variable
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount EFS to HTML directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and necessary extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html

# Download and set up WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo vi /var/www/html/wp-config.php

# Restart Apache
sudo service httpd restart

```

## Auto Scaling Group Launch Template Script

```

\`\`\`bash
#!/bin/bash

# Update software packages
sudo yum update -y

# Install Apache web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and necessary extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set environment variable
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# Mount EFS to HTML directory
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
\`\`\`
```
