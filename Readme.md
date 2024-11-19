# Keycloak SSO Integration with ArgoCD and Harbor

## Overview
This repository provides a complete guide for implementing Single Sign-On (SSO) authentication using Keycloak with ArgoCD and Harbor on a Kubernetes cluster. The setup uses AWS EKS as the Kubernetes platform.

## Features
- Keycloak SSO integration
- ArgoCD deployment and configuration
- Harbor registry setup
- Nginx Ingress Controller
- Automatic TLS certificate management with cert-manager
- Terraform-based EKS cluster provisioning

## Prerequisites
- AWS CLI installed
- Terraform installed
- Helm installed
- A domain name (for configuring subdomains)
- Kubernetes knowledge

## Quick Start

### 1. Cluster Provisioning
cd kubernetes-cluster
terraform init --upgrade
terraform apply --auto-approve
Configure kubectl:
aws eks update-kubeconfig --region us-east-1 --name realworld-cluster
### 2. Infrastructure Setup

#### Install Nginx Ingress
kubectl apply -f ingress-nginx.yml
#### Install Cert Manager
```helm repo add jetstack https://charts.jetstack.io --force-update```
```helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.15.2 \
  --set crds.enabled=true```
#### Create Namespaces and Certificates
kubectl apply -f cert-manager/namespace.yml
kubectl apply -f cert-manager/certificate.yml
### 3. Application Deployment

#### Install ArgoCD
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade --install argo-cd argo/argo-cd \
  --version 7.3.10 \
  --reuse-values \
  -f argocd-values.yml \
  --namespace argocd \
  --create-namespace
#### Install Keycloak
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade --install keycloak bitnami/keycloak \
  --reuse-values \
  -f keycloak-values.yml \
  --namespace keycloak \
  --create-namespace
#### Install Harbor
helm repo add harbor https://helm.goharbor.io
helm upgrade --install harbor harbor/harbor \
  --namespace harbor \
  --create-namespace \
  --reuse-values \
  -f harbor-values.yml
## Configuration

### Domain Setup
Configure the following subdomains in your DNS:
- argocd.your-domain.com
- keycloak.your-domain.com
- harbor.your-domain.com

### SSO Configuration

#### ArgoCD SSO Setup
1. Create a new client in Keycloak
2. Configure client settings and scopes
3. Update ArgoCD configuration:
kubectl edit secret argocd-secret -n argocd
kubectl edit configmap argocd-cm -n argocd
kubectl edit configmap argocd-rbac-cm -n argocd
#### Harbor SSO Setup
1. Create a new client in Keycloak
2. Configure Harbor authentication settings through UI
3. Test SSO login

## Cleanup

Remove all installations:
# Uninstall Harbor
helm uninstall harbor -n harbor

# Uninstall ArgoCD
helm uninstall argo-cd --namespace argocd

# Uninstall Keycloak
helm uninstall keycloak --namespace keycloak

# Remove Nginx Ingress
kubectl delete -f ingress-nginx.yaml

# Destroy cluster
cd kubernetes-cluster
terraform destroy --auto-approve
## Default Credentials

### Keycloak
- Username: admin
- Password: admin12345

### Harbor
- Username: admin
- Password: Harbor12345

## Repository Structure
.
├── kubernetes-cluster/     # Terraform configuration files
├── cert-manager/          # Certificate configuration
├── argocd-values.yml      # ArgoCD Helm values
├── harbor-values.yml      # Harbor Helm values
├── keycloak-values.yml    # Keycloak Helm values
└── ingress-nginx.yml      # Nginx ingress configuration
## Contributing
Feel free to submit issues and enhancement requests.

## License
[License information