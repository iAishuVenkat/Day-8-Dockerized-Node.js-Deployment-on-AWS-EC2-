# Docker to EC2 Deployment Guide

## Prerequisites

### Local Machine Requirements
- SSH client
- .pem key file from AWS

### AWS Requirements
- AWS account
- EC2 instance running (Amazon Linux 2)
- EC2 key pair (.pem file)
- Security group configured

## Step 1: Prepare Your EC2 Instance

### 1.1 Launch EC2 Instance
- Go to AWS Console > EC2 > Launch Instance
- Choose: Amazon Linux 2 AMI
- Instance type: t2.micro (free tier)
- Create or select an existing key pair (download the .pem file)
- Configure security group (see below)

### 1.2 Configure Security Group
Allow inbound traffic on:
- Port 22 (SSH) from your IP
- Port 80 (HTTP) from anywhere (0.0.0.0/0)

### 1.3 Connect to EC2 and Install Docker
```bash
# Connect to your EC2 instance (use full path to your .pem file)
ssh -i "C:\Users\YourUsername\Downloads\your-key.pem" ec2-user@your-ec2-public-ip

# Update system
sudo yum update -y

# Install Docker
sudo yum install -y docker

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add ec2-user to docker group
sudo usermod -a -G docker ec2-user

# Exit and reconnect for group changes to take effect
exit
```

Reconnect:
```bash
ssh -i "C:\Users\YourUsername\Downloads\your-key.pem" ec2-user@your-ec2-public-ip
```

## Step 2: Copy Project Files to EC2

Run this from your local project directory:
```bash
# Use full path to your .pem file
scp -i "C:\Users\YourUsername\Downloads\your-key.pem" -r . ec2-user@your-ec2-public-ip:~/my-app
```

Note: If you get "Permission denied" on Windows, fix key permissions first:
```powershell
icacls "C:\Users\YourUsername\Downloads\your-key.pem" /inheritance:r /grant:r "%USERNAME%:R"
```

## Step 3: Build Docker Image on EC2

```bash
# Connect to EC2
ssh -i "C:\Users\YourUsername\Downloads\your-key.pem" ec2-user@your-ec2-public-ip

# Go to app folder
cd ~/my-app

# Build Docker image
docker build -t my-docker-app .
```

## Step 4: Run the Container

```bash
docker run -d --name my-app -p 80:3000 --restart unless-stopped my-docker-app
```

This maps host port 80 to container port 3000.

## Step 5: Verify Deployment

```bash
# Check container is running
docker ps

# Check logs
docker logs my-app

# Test on EC2 (use port 80, not 3000)
curl http://localhost
```

Then open your browser:
```
http://your-ec2-public-ip
```

Expected response:
```json
{"message":"Hello from Docker on EC2!","timestamp":"..."}
```

## Management Commands

```bash
# Stop container
docker stop my-app

# Start container
docker start my-app

# Remove container
docker stop my-app && docker rm my-app

# View live logs
docker logs -f my-app
```

## Troubleshooting

### .pem file not found
- Always use the full path: `"C:\Users\YourUsername\Downloads\your-key.pem"`
- Find your .pem file:
  ```powershell
  Get-ChildItem -Path C:\Users\$env:USERNAME -Recurse -Filter "*.pem" 2>$null
  ```

### Permission denied (publickey)
- Fix key permissions on Windows:
  ```powershell
  icacls "C:\Users\YourUsername\Downloads\your-key.pem" /inheritance:r /grant:r "%USERNAME%:R"
  ```

### curl http://localhost:3000 fails
- Container is mapped to port 80, not 3000
- Use `curl http://localhost` instead

### Connection refused in browser
- Check security group allows port 80 inbound
- Verify container is running: `docker ps`

### Port already in use
```bash
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```
