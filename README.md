## CI/CD Pipeline – Auto Deploy Flask App to AWS EC2 Using GitHub Actions
# Automated CI/CD for Your Flask App on AWS EC2 🚀
---
In this repositery i have made a pipeline using github actions to deploy my simple dockerized flask app to my AWS EC2 every time when i changes to my `main` branch.
----

##  1. EC2 Instance Preparation (One-Time Setup)

First we need to prepare EC2 instance.

**Launch an Ubuntu EC2 Instance:**

If you haven't already, launch an Ubuntu EC2 instance from the AWS console.

**Connect to Your EC2 Instance via SSH and Run These Commands:**

```bash
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
