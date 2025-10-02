# 1. Create EKS cluster with Fargate

eksctl create cluster --name demo-cluster --region us-east-1 --fargate

# 2. Update kubeconfig
aws eks update-kubeconfig --name demo-cluster --region us-east-1

# 3. Deploy app (2048 game)
kubectl apply -f deploy.yaml
kubectl apply -f service.yaml

# 4. Create Fargate profile
eksctl create fargateprofile \
  --cluster demo-cluster \
  --region us-east-1 \
  --name alb-sample-app \
  --namespace game-2048

# 5. Associate IAM OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --approve

# 6. Create IAM policy for ALB Controller
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

# 7. Create IAM Service Account for ALB Controller
eksctl create iamserviceaccount \
  --cluster demo-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

# 8. Install AWS Load Balancer Controller via Helm
helm repo add eks https://aws.github.io/eks-charts
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<your-vpc-id>

# 9. Verify deployment
kubectl get deployment -n kube-system aws-load-balancer-controller
