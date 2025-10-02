# Deploying 2048 Game on AWS EKS with Fargate & ALB

This project demonstrates deploying a containerized 2048 game on Amazon EKS using:
- Fargate for serverless pods
- AWS Load Balancer Controller for ingress
- IAM OIDC integration for secure pod access

## Steps

1. Create EKS cluster with Fargate
2. Deploy 2048 game (Deployment + Service)
3. Create Fargate profile for the `game-2048` namespace
4. Install and configure AWS Load Balancer Controller
5. Expose application using ALB Ingress

