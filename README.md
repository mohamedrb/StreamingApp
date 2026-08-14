# StreamingApp

### Step 1 — Fork the repository

### Step 2 - Create Dockerfiles
      StreamingApp Frontend
        ↓
      streamingapp-frontend
      
      StreamingApp Backend
              ↓
      streamingapp-backend

### Step 3 Build Docker images
      docker build -t streamingapp-frontend:1.0 ./frontend

      docker build -t streamingapp-backend:1.0 ./backend

### Step 4 Test Docker containers

docker run -d `
  --name streamingapp-backend `
  -p 5000:5000 `
  streamingapp-backend:1.0

### Step 5 Create ECR repositories

aws ecr create-repository `
  --repository-name streamingapp-frontend `
  --region ap-south-1

aws ecr create-repository `
  --repository-name streamingapp-backend `
  --region ap-south-1

### Step 6 Authenticate Docker with ECR

aws sts get-caller-identity

aws ecr get-login-password --region ap-south-1 |
docker login --username AWS --password-stdin YOUR_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com

### Step 7 Tag images

docker tag streamingapp-frontend:1.0 `
YOUR_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-frontend:1.0

docker tag streamingapp-backend:1.0 `
YOUR_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-backend:1.0

docker push `
YOUR_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-frontend:1.0

docker push `
YOUR_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-backend:1.0

### Step 8 AWS CLI

### Step 9 Create EKS

eksctl create cluster `
  --name streamingapp-cluster `
  --region ap-south-1 `
  --nodes 2 `
  --node-type t3.medium

### Step 10 Connect kubectl to EKS

aws eks update-kubeconfig `
  --region ap-south-1 `
  --name streamingapp-cluster

### Step 11 Create Helm char

### Step 12 Jenkins

### Step 13 CloudWatch monitoring

aws eks create-addon `
  --cluster-name streamingapp-cluster `
  --addon-name amazon-cloudwatch-observability `
  --configuration-values '{"otelContainerInsights":{"enabled":true}}'

  aws eks describe-addon `
  --cluster-name streamingapp-cluster `
  --addon-name amazon-cloudwatch-observability `
  --query "addon.status" `
  --output text

