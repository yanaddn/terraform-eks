# terraform-eks-demo

## Overview

Creates an EKS cluster on AWS (`eu-west-3`) with Terraform:

- `vpc.tf` – VPC with 3 private and 3 public subnets (one per AZ) and a single NAT gateway, using the `terraform-aws-modules/vpc/aws` module
- `eks-cluster.tf` – EKS cluster (Kubernetes 1.35) with a managed node group of `t3.micro` nodes, using the `terraform-aws-modules/eks/aws` module
- `versions.tf` – Terraform and AWS provider version constraints
- `terraform.tfvars` – CIDR ranges for the VPC and subnets
- `nginx-config.yaml` – nginx Deployment plus a LoadBalancer Service to test the cluster

## CLI commands

### initialize (add `-upgrade` after changing module/provider versions)

    terraform init

### preview terraform actions

    terraform plan

### apply configuration (`terraform.tfvars` is loaded automatically)

    terraform apply

### connect kubectl to the cluster

    aws eks update-kubeconfig --region eu-west-3 --name myapp-eks-cluster
    kubectl get nodes

### deploy nginx and get the load balancer address

    kubectl apply -f nginx-config.yaml
    kubectl get svc nginx

Open `http://<EXTERNAL-IP>` in the browser (allow a few minutes for health checks).

### delete nginx before destroying (removes the load balancer Kubernetes created)

    kubectl delete -f nginx-config.yaml

### destroy everything from tf files

    terraform destroy

### show resources and components from current state

    terraform state list

### show current state of a specific resource/data

    terraform state show module.myapp-vpc.aws_vpc.this[0]

### for debugging in TF

    export TF_LOG=TRACE

## Final outcome

A running EKS cluster with nginx reachable through an AWS load balancer.
