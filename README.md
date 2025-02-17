# OpenTelemetry E-Commerce Microservices Demo
This project demonstrates the implementation of a distributed e-commerce application using 20 microservices. The application is deployed on AWS EKS (Elastic Kubernetes Service) and includes a full CI/CD pipeline using GitHub Actions and GitOps with ArgoCD. Additionally, the AWS Load Balancer (ALB) Ingress Controller is implemented to manage ingress traffic.

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

## Local Development

1. Clone the repository: <br>
`git clone https://github.com/imran99744/ultimate-devops-project-demo.git` <br>
`cd ultimate-devops-project-demo`

2. Start the application locally:
   `docker compose up -d`

3. Access the application at `http://localhost:8080`

## Cloud Provisioning with Terraform

`cd terraform`
`cd backend`

Initialize Terraform

`terraform init` and then `terraform apply`

Get back to the Terraform root

`cd ..`

Initialize all of the Terraform configs

`terraform init`

Apply the Terraform configuration to provision AWS resources:

`terraform apply`

## Deploying to AWS EKS
Configure kubectl to connect to your EKS cluster:

`aws eks --region <region> update-kubeconfig --name <cluster-name>`

Deploy the microservices:

`cd kubernetes`

`kubectl apply -f complete-deploy.yaml`


Set up a custom domain using Route 53 or another DNS provider.

# How to setup alb add on

## Setup OIDC Connector
commands to configure IAM OIDC provider

`export cluster_name=demo-cluster`
`oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)`


## Check if there is an IAM OIDC provider configured already
  - aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4\n
If not, run the below command

`eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve`


## Download IAM policy
`curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json`

Create IAM Policy

`aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json`

Create IAM Role

`eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve`

## Deploy ALB controller
Add helm repo

`helm repo add eks https://aws.github.io/eks-charts`
Update the repo

`helm repo update eks`

Install

`helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>`

Verify that the deployments are running.

`kubectl get deployment -n kube-system aws-load-balancer-controller`

You might face the issue, unable to see the loadbalancer address while giving k get ing -n robot-shop at the end. To avoid this your AWSLoadBalancerControllerIAMPolicy should have the required permissions for elasticloadbalancing:DescribeListenerAttributes.

## Run the following command to retrieve the policy details and look for elasticloadbalancing:DescribeListenerAttributes in the policy document.
`aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.DefaultVersionId' --output text)`

If the required permission is missing, update the policy to include it

## Download the current policy
`aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.DefaultVersionId' --output text) \
    --query 'PolicyVersion.Document' --output json > policy.json`

## Edit policy.json to add the missing permissions
`{
  "Effect": "Allow",
  "Action": "elasticloadbalancing:DescribeListenerAttributes",
  "Resource": "*"
}`

## Create a new policy version
`aws iam create-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://policy.json \
    --set-as-default`

