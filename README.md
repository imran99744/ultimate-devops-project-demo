# OpenTelemetry E-Commerce Microservices Demo
This project demonstrates the implementation of a distributed e-commerce application using 20 microservices. The application is deployed on AWS EKS (Elastic Kubernetes Service) and includes a full CI/CD pipeline using GitHub Actions and GitOps with ArgoCD.

## Key Features
- 20 Microservices: Simulates a real-world e-commerce platform with services written in Go, .NET, Python, and more.

- AWS EKS Deployment: Fully automated provisioning and deployment on AWS using Terraform.

- Custom Domain: Configured a custom domain for the application.

- CI/CD Pipeline: Automated build, test, and deployment using GitHub Actions.

- GitOps with ArgoCD: Continuous Delivery using GitOps principles.

## Project Overview
This project was implemented in the following phases:

1. Local Development:

      - Ran the application locally using Docker Compose.

      - Explored Dockerfiles for Go, .NET, and Python applications to understand containerization.

2. Cloud Provisioning:

      - Provisioned AWS cloud resources (EKS, VPC, IAM, etc.) from scratch using Terraform.

3. Kubernetes Deployment:

      - Deployed the 20 microservices on AWS EKS.

      - Configured a custom domain for the application.

4. CI/CD Automation:

      - Implemented a CI/CD pipeline using GitHub Actions.

      - Used ArgoCD for GitOps-based Continuous Delivery.

# Getting Started
## Prerequisites

- Docker and Docker Compose
- Terraform
- AWS CLI with proper credentials
- kubectl and eksctl
