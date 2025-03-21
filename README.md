AWS Multi-Region EKS Deployment with CI/CD, Disaster Recovery, and Monitoring
Project Overview
This project sets up a highly available, scalable, and secure AWS EKS (Elastic Kubernetes Service) cluster across multiple regions. It includes CI/CD automation, disaster recovery, and monitoring using Prometheus & Grafana to ensure resilience and observability.

Table of Contents
Architecture Overview
Prerequisites
Step 1: Multi-Region EKS Deployment
Step 2: CI/CD with GitHub Actions & ArgoCD
Step 3: Disaster Recovery Setup
Step 4: Monitoring & Alerting with Prometheus & Grafana
Accessing the Cluster & Applications
Cleaning Up Resources
Architecture Overview
This setup includes:
✅ EKS clusters deployed across multiple AWS regions
✅ CI/CD pipeline using GitHub Actions & ArgoCD
✅ Disaster Recovery using AWS Backup & Cross-Region Replication
✅ Monitoring & Alerting with Prometheus & Grafana

Prerequisites
Before proceeding, ensure you have:

AWS CLI installed and configured (aws configure)
kubectl installed for cluster management
eksctl installed for EKS provisioning
Terraform installed
GitHub Actions set up for CI/CD
Docker installed for container builds
Step 1: Multi-Region EKS Deployment
1️⃣ Deploy EKS Clusters Using CloudFormation
Run the following CloudFormation template to deploy an EKS cluster in us-east-1 and us-west-2:

aws cloudformation create-stack --stack-name eks-cluster-primary \
    --template-body file://eks-cluster.yaml \
    --region us-east-1
Repeat for the secondary region:

aws cloudformation create-stack --stack-name eks-cluster-secondary \
    --template-body file://eks-cluster.yaml \
    --region us-west-2
2️⃣ Configure kubectl to Use the Cluster

aws eks --region us-east-1 update-kubeconfig --name eks-cluster-primary
aws eks --region us-west-2 update-kubeconfig --name eks-cluster-secondary
Step 2: CI/CD with GitHub Actions & ArgoCD
1️⃣ Deploy ArgoCD to EKS

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
2️⃣ Expose ArgoCD Server

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
Retrieve the external IP:


kubectl get svc -n argocd
3️⃣ Configure GitHub Actions
Create a GitHub Actions workflow (.github/workflows/deploy.yml):

name: Deploy to EKS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to EKS
        run: |
          aws eks --region us-east-1 update-kubeconfig --name eks-cluster-primary
          kubectl apply -f k8s-manifests/
Push changes to GitHub, and the workflow will deploy applications automatically.

Step 3: Disaster Recovery Setup
1️⃣ Enable AWS Backup

aws backup create-backup-plan --backup-plan file://backup-plan.json
2️⃣ Configure Cross-Region Replication

aws s3api put-bucket-replication --bucket my-backup-bucket \
    --replication-configuration file://replication.json
Step 4: Monitoring & Alerting with Prometheus & Grafana
1️⃣ Deploy Prometheus & Grafana

kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
2️⃣ Expose Grafana

kubectl patch svc prometheus-grafana -n monitoring -p '{"spec": {"type": "LoadBalancer"}}'
Retrieve the external IP:

kubectl get svc -n monitoring
Accessing the Cluster & Applications
1️⃣ Access ArgoCD
Retrieve the initial password:

kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
Access the UI at:

http://<ARGOCD-EXTERNAL-IP>
2️⃣ Access Grafana
Retrieve the credentials:

kubectl get secret prometheus-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
Access the Grafana dashboard at:

http://<GRAFANA-EXTERNAL-IP>
Cleaning Up Resources
To remove all AWS resources, run:

aws cloudformation delete-stack --stack-name eks-cluster-primary --region us-east-1
aws cloudformation delete-stack --stack-name eks-cluster-secondary --region us-west-2
To delete ArgoCD:

kubectl delete ns argocd
To delete monitoring stack:

kubectl delete ns monitoring
Conclusion
This project successfully deployed a multi-region, highly available EKS cluster with:

✅ CI/CD using GitHub Actions & ArgoCD
✅ Disaster Recovery with AWS Backup & Cross-Region Replication
✅ Monitoring with Prometheus & Grafana

🚀 Your infrastructure is now fully automated, resilient, and production-ready!

