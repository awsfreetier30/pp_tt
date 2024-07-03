Scope of the project:

This project demonstrates the setup and deployment of a static website using GitHub Actions for Continuous Integration and Continuous Deployment (CI/CD) to an Amazon Web Services (AWS) EC2 instance. It covers version control, automated deployment workflows, cloud infrastructure setup, and web server configuration.

Pre-requisites and Technology Stack:

Type: Intermediate

Technologies used:

•	Git and GitHub for version control
•	GitHub Actions for CI/CD
•	AWS EC2 for hosting
•	Apache web server
•	Bash scripting
•	SSH for secure connections

Pre-requisites:

•	Basic understanding of Git and GitHub
•	Familiarity with cloud concepts and AWS
•	Knowledge of Linux command line
•	Understanding of web servers and static websites
Step-by-step approach:
•	Set up local environment and repository
•	Create GitHub Actions workflow
•	Set up AWS EC2 instance
•	Configure EC2 instance
•	Configure GitHub repository secrets
•	Set up Git authentication
•	Push changes to GitHub
•	Monitor deployment
•	Verify deployment
•	Troubleshoot if necessary
•	Update website as needed






# Static Website Deployment with GitHub Actions and AWS EC2

This project demonstrates how to set up and deploy a static website using GitHub Actions for Continuous Integration and Continuous Deployment (CI/CD) to an Amazon Web Services (AWS) EC2 instance.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Setup Instructions](#setup-instructions)
4. [Deployment Process](#deployment-process)
5. [Troubleshooting](#troubleshooting)
6. [Updating the Website](#updating-the-website)

## Project Overview

This project covers:
- Setting up a GitHub repository for a static website
- Configuring GitHub Actions for automated deployment
- Launching and configuring an AWS EC2 instance
- Setting up Apache web server on EC2
- Implementing secure file transfer and server management

## Prerequisites

- Git and GitHub account
- AWS account with EC2 access
- Basic understanding of:
  - Git version control
  - Linux command line
  - Web servers and static websites
  - Cloud computing concepts

## Setup Instructions

### 1. Local Environment Setup

```bash
git clone https://github.com/awsfreetier30/code-test-upwork.git
cd code-test-upwork
mkdir -p .github/workflows
echo "private_key" > .gitignore
echo ".env" >> .gitignore
```

### 2. GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Deploy to EC2
      env:
        PRIVATE_KEY: ${{ secrets.EC2_SSH_PRIVATE_KEY }}
        HOST: ${{ secrets.EC2_HOST }}
        USER: ubuntu
      run: |
        echo "$PRIVATE_KEY" > private_key
        chmod 600 private_key
        ssh -i private_key -o StrictHostKeyChecking=no ${USER}@${HOST} '
          sudo chmod -R 755 /var/www/html
          sudo chown -R ubuntu:ubuntu /var/www/html
        '
        scp -i private_key -o StrictHostKeyChecking=no -r ./* ${USER}@${HOST}:/var/www/html/
        ssh -i private_key -o StrictHostKeyChecking=no ${USER}@${HOST} '
          sudo systemctl restart apache2
        '
```

### 3. AWS EC2 Setup

- Launch an EC2 instance with Ubuntu 22.04 AMI
- Configure security groups for SSH (port 22) and HTTP (port 80)

### 4. EC2 Configuration

SSH into your EC2 instance and run:

```bash
#!/bin/bash

sudo apt update
sudo apt upgrade -y
sudo apt install -y apache2
sudo a2enmod rewrite

sudo tee /etc/apache2/sites-available/000-default.conf > /dev/null <<EOT
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOT

sudo chown -R ubuntu:ubuntu /var/www/html
sudo chmod -R 755 /var/www/html
sudo usermod -a -G ubuntu www-data
sudo systemctl restart apache2

echo "EC2 instance setup complete!"
```

### 5. GitHub Repository Configuration

Add the following secrets to your GitHub repository:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `EC2_SSH_PRIVATE_KEY`
- `EC2_HOST`

### 6. Git Authentication

```bash
git config --global credential.helper store
git config --global user.name "Your GitHub Username"
git config --global user.email "your.email@example.com"
git remote set-url origin https://YOUR-USERNAME:YOUR-TOKEN@github.com/awsfreetier30/code-test-upwork.git
```

## Deployment Process

1. Push changes to GitHub:
   ```bash
   git add .
   git commit -m "Update website content"
   git push origin main
   ```

2. Monitor the GitHub Actions workflow in the "Actions" tab of your repository.

3. Verify deployment by visiting your EC2 instance's public DNS or IP address.

## Troubleshooting

- Check GitHub Actions logs for error messages
- SSH into EC2 and check Apache logs:
  ```
  sudo tail -f /var/log/apache2/error.log
  ```
- Verify file permissions on EC2:
  ```
  sudo chown -R ubuntu:ubuntu /var/www/html
  sudo chmod -R 755 /var/www/html
  ```

## Updating the Website

1. Make changes to your local files
2. Commit and push changes to GitHub
3. GitHub Actions will automatically deploy updates to EC2

For any issues or questions, please open an issue in the GitHub repository.


