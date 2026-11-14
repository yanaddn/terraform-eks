# Terraform & EKS project

Terraform setup for an AWS EKS cluster with a custom VPC, managed node group, and an nginx LoadBalancer demo.

- [Overview](#overview)
- [CLI Commands](#cli-commands)
- [Final Outcome](#final-outcome)

## Overview

- [x] Created the VPC by using the VPC module
- [x] Created the EKS cluster and worker nodes by using the EKS module
- [x] Configured Kubernetes provider to authenticate with K8s cluster
- [x] Applied configurations
- [x] Deployed nginx Application/Pod

<img width="1384" height="853" alt="image" src="https://github.com/user-attachments/assets/3b39bec0-155d-4463-8b56-e00c50df7e7b" />

An EKS cluster on AWS (`eu-west-3`) with Terraform:
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

A running EKS cluster with nginx reachable through an AWS load balancer:

### EKS cluster 
<img width="1578" height="848" alt="image" src="https://github.com/user-attachments/assets/baee9602-94ee-4b8d-b9ac-15c4c03e3834" />

```
% kubectl get node
NAME                                       STATUS   ROLES    AGE     VERSION
ip-10-0-1-33.eu-west-3.compute.internal    Ready    <none>   5m15s   v1.35.9-eks-8418f54
ip-10-0-2-119.eu-west-3.compute.internal   Ready    <none>   4m      v1.35.9-eks-8418f54
ip-10-0-3-148.eu-west-3.compute.internal   Ready    <none>   5m32s   v1.35.9-eks-8418f54
```
<img width="1576" height="821" alt="image" src="https://github.com/user-attachments/assets/d86fade0-c0e1-4b5d-8db8-94cdf5cebc71" />

<img width="1577" height="670" alt="image" src="https://github.com/user-attachments/assets/aef4b61a-c0f9-4c41-8070-e46d7b7e385a" />

### Load balancer
```
% kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)        AGE
kubernetes   ClusterIP      172.20.0.1      <none>                                                                    443/TCP        100m
nginx        LoadBalancer   172.20.246.47   ab715384d7a77425daa7d55f4f95c072-1787603864.eu-west-3.elb.amazonaws.com   80:32423/TCP   7m57s
```
<img width="1578" height="842" alt="image" src="https://github.com/user-attachments/assets/9c530398-dcee-4a4f-8206-114a58d39fea" />

### Nginx 
```
% kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-75fdcbbc74-s7m89   1/1     Running   0          17m
```
<img width="1267" height="690" alt="image" src="https://github.com/user-attachments/assets/6c46c9dc-84bb-4e73-8da1-929d463b7bdf" />

