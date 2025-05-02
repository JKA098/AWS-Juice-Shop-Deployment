# 🍹 AWS Juice Shop Deployment via VPC (Private EC2 + Bastion + Docker)
![AWS](https://img.shields.io/badge/platform-AWS-orange)
![Docker](https://img.shields.io/badge/containerized-Docker-blue)
![Status](https://img.shields.io/badge/status-Stable-brightgreen)
![Security](https://img.shields.io/badge/security-Bastion--Hardened-lightgrey)

## 🌞 Overview
This guide documents the full process of setting up a secure AWS environment to run OWASP Juice Shop from a **private EC2 instance**, accessed via a **public EC2 bastion host** using an **SSH tunnel**. The app runs in **Docker** on the private instance.

---

## 🧱 Architecture Overview

- 🟢 **Public Subnet**: Bastion EC2 (SSH only)
- 🔵 **Private Subnet**: Juice Shop EC2 (runs in Docker)
- 🌐 **SSH Tunnel**: Bastion ➜ Private EC2
- 🐳 **Dockerized App**: Juice Shop running in a container[Optional]

## ✅ Prerequisites
- AWS account
- SSH client (e.g., PowerShell with OpenSSH, or Unix-based terminal)
- `.pem` key file downloaded and stored securely (see next section to generate it)
- 🐳 [Optional] **Docker Desktop** for local testing

---

## 🔐 Creating a .pem Key Pair in AWS

### 🧭 Location:
**AWS Console → EC2 → Key Pairs**

### 🪜 Steps:
1. **Go to the EC2 Dashboard**
2. On the left menu, click **Key Pairs**
3. Click **Create Key Pair**
4. In the popup:
   - **Name**: `tech-one-ec2-key` (or your preferred name)
   - **Key pair type**: `RSA`
   - **Private key format**: `pem`
5. Click **Create key pair**

✅ The `.pem` file will download automatically.

> 🔒 **Important**: AWS only shows the private key once. Store it securely (e.g., `~/Documents/AWS/`).

### 🖥️ Use the `.pem` File to SSH:
```bash
chmod 400 path/to/your-key.pem
ssh -i path/to/your-key.pem ubuntu@<public-ec2-ip>
```

# AWS Juice Shop Deployment Steps

## 1️⃣ Create a Custom VPC
- **VPC CIDR**: `10.0.0.0/16`

### Create Subnets
- **Public Subnet**: `10.0.1.0/24` → `public-subnet-test`  
- **Private Subnet**: `10.0.2.0/24` → `private-subnet-test`

---

## 2️⃣ Internet Gateway
- Create and attach an **Internet Gateway** to your VPC

### Public Subnet Route Table
- Add route: `0.0.0.0/0` → **Internet Gateway** (`igw-xxxxxxxx`)
- Associate this route table with the **public subnet**

---

## 3️⃣ NAT Gateway
- **Allocate** an Elastic IP
- **Create** a NAT Gateway in the **public subnet** (not private)
- **Attach** the Elastic IP

### Private Subnet Route Table
- Add route: `0.0.0.0/0` → **NAT Gateway** (`nat-xxxxxxxx`)
- Associate this route table with the **private subnet**

---

## 4️⃣ Security Groups

### Public EC2 (Bastion Host) Inbound Rules:
- SSH: TCP 22 from **Your IP**
- HTTP (optional): TCP 80 from `0.0.0.0/0`
- Juice Shop: TCP 3000 from `0.0.0.0/0`

### Private EC2 (Juice Shop Host) Inbound Rules:
- SSH: TCP 22 from `10.0.1.0/24` (Bastion Host subnet)
- Custom TCP: TCP 3000 from `10.0.0.0/16` (or Bastion's subnet)

- **Outbound**: Allow all (default)

---

## 5️⃣ Launch EC2 Instances

### Public EC2 (Bastion Host)
- Subnet: **Public**
- Auto-assign Public IP: **Yes**
- Security Group: **Bastion SG**

### Private EC2 (App Host)
- Subnet: **Private**
- Auto-assign Public IP: **No**
- Security Group: **Juice Shop SG**

---

## 6️⃣ Install Docker on Private EC2

SSH into the private EC2 via the bastion (using SSH port forwarding), then run:

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ${USER}
sudo docker run -d -p 3000:3000 bkimminich/juice-shop

```

## 7️⃣ Run Juice Shop in Docker

```bash
sudo docker run -d -p 3000:3000 bkimminich/juice-shop
```

Verify it’s running:

```bash
sudo docker ps
```

## 8️⃣ Set Up SSH Tunnel from Local Machine

```bash
ssh -i "C:\\path\\to\\your-key.pem" -L 3000:<private-ec2-ip>:3000 ubuntu@<public-ec2-ip>
```

Access in Browser:
```bash
http://localhost:3000
```
