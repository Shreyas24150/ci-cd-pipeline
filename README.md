# CI/CD Pipeline – Auto Deploy Flask App to AWS EC2 Using GitHub Actions

---
In this repositery i have made a pipeline using github actions to deploy my simple dockerized flask app to my AWS EC2 every time when i changes to my `main` branch.
---

##  1. EC2 Instance Preparation (One-Time Setup)

First we need to prepare EC2 instance.

**Launch an Ubuntu EC2 Instance:**

Launch an Ubuntu EC2 instance from the AWS console.

**Connect to Your EC2 Instance via SSH and Run These Commands:**

# Update your package list
sudo apt update

# Install Docker
sudo apt install docker.io -y

# Add the 'ubuntu' user to the 'docker' group to run Docker commands without sudo
sudo usermod -aG docker ubuntu

# Enable Docker to start on boot
sudo systemctl enable docker

# Install Git (we'll need this later, though not directly in the GitHub Action)
sudo apt install git

**Crucial: Open Your Ports!**
Make sure EC2 instance's security group allows inbound traffic on:

  Port 22 (SSH): For secure shell access.
  Port 5000 (Flask): This is the default port Flask app will run on.

---

## 2. Set Up Your Repository for CI/CD

**project structure**
```bash
/your-repo
├── app.py              # Flask application code
├── requirements.txt    # Python dependencies for Flask app
├── Dockerfile          # Instructions to build Docker image
├── docker-compose.yml  # Defines Flask service for Docker Compose
└── .github/workflows/  # This is where our CI/CD workflow lives
    └── deploy.yml      # GitHub Actions workflow file

---

## 3. Add the deploy.yml Workflow

name: Deploy to EC2

on:
  push:
    branches:
      - main # This workflow triggers on pushes to the 'main' branch

jobs:
  deploy:
    runs-on: ubuntu-latest # The type of runner that the job will run on

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3 # Checks out repository under $GITHUB_WORKSPACE

    - name: Copy files to EC2
      uses: appleboy/scp-action@master  # Action to securely copy files to EC2
      with:
        host: ${{ secrets.EC2_HOST }}   # EC2 public IP (from GitHub Secrets)
        username: ubuntu                # Default username for Ubuntu EC2 instances
        key: ${{ secrets.EC2_KEY }}     # EC2 private key (from GitHub Secrets)
        source: "."                     # Copy everything from the current directory
        target: "/home/ubuntu/app"      # Target directory on EC2 instance

    - name: SSH into EC2 & Run Docker Compose
      uses: appleboy/ssh-action@master  # Action to execute commands via SSH on EC2
      with:
        host: ${{ secrets.EC2_HOST }}   # EC2 public IP
        username: ubuntu                # Default username for Ubuntu EC2 instances
        key: ${{ secrets.EC2_KEY }}     # EC2 private key
        script: |                       # Commands to execute on EC2
          cd /home/ubuntu/app           # Navigate to application directory
          docker compose down           # Stop any running Docker Compose services
          docker compose up --build -d  # Build images (if needed) and start services in detached mode

---

## 4. Add GitHub Secrets

 1. Go to your GitHub repository.
 2. Navigate to Settings > Secrets and variables > Actions.
 3. Click on New repository secret.

**need to create two secrets:**
EC2_HOST : EC2 Instance's Public IP Address.
EC2_KEY : contents of EC2 private key file (.pem file)
    How to EC2 private key : open your-key.pem file copy that content and paste it to EC2_KEY

---

## 5. Push Your Code to main → Auto Deploy!

```bash
git add .
git commit -m "Set up automated CI/CD deployment"
git push origin main
