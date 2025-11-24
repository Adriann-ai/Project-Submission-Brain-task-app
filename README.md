🚀 CI/CD with AWS CodePipeline + ArgoCD on EKS

A complete GitOps workflow using CodePipeline for CI and Argo CD for CD, deployed on Amazon EKS.

📚 What This Project Covers

        ✅ CI Pipeline using AWS CodePipeline + CodeBuild
        
        ✅ Containerization using Docker & ECR
        
        ✅ CD Pipeline using Argo CD (GitOps)
        
        ✅ Kubernetes deployment on Amazon EKS
        
        ✅ Full automation from GitHub → ECR → EKS → Argo CD

🛠️ Step-by-Step Implementation
🔹 Step 1 — Push Source Code to GitHub

Push your application source code from the local machine to your remote GitHub repository.

🔹 Step 2 — Create Dockerfile & Build Local Image

Use an Apache HTTPD base image and copy your build output to the Apache web root.

Dockerfile
      FROM httpd:latest
      COPY dist/ /usr/local/apache2/htdocs
      EXPOSE 80

Build & Test the Docker Image Locally
      docker build -t brain-task:latest .
      docker run -itd --name brain-task -p 3000:80 brain-task:latest


👉 Access locally via: http://localhost:3000

🔹 Step 3 — Create Buildspec for CodeBuild
buildspec.yaml
                  version: 0.2
                  
                  env:
                    variables:
                      IMAGE_REPO: "300615130618.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app"
                      IMAGE_TAG: "latest"
                  
                  phases:
                    install:
                      commands:
                        - apt update -y
                        - apt install -y docker.io
                  
                    pre_build:
                      commands:
                        - echo "Amazon ECR login"
                        - aws ecr get-login-password --region ap-south-1 | \
                          docker login --username AWS --password-stdin $IMAGE_REPO
                  
                    build:
                      commands:
                        - echo "Building Docker image..."
                        - docker build -t brain-task-app .
                        - docker tag brain-task-app:latest $IMAGE_REPO:$IMAGE_TAG
                  
                    post_build:
                      commands:
                        - echo "Pushing Docker image to ECR..."
                        - docker push $IMAGE_REPO:$IMAGE_TAG

🔹 Step 4 — Create Kubernetes Manifests

Create these files:

🧩 deployment.yaml

🌐 service.yaml

These will be used by ArgoCD for CD part

🔹 Step 5 — Create AWS ECR Repository

       This is where CodeBuild will push the Docker images.

🔹 Step 6 — Configure CodeBuild & CodePipeline (CI)

                Attach correct IAM role
                
                Use buildspec.yaml
                
                Connect repository to GitHub

          Pipeline flow:
          
               GitHub → CodeBuild → ECR

🔹 Step 7 — Create EKS Cluster + Node Group

                Configure:
                
                      EKS cluster
                      
                      Node group
                      
                      IAM permissions
                      
                      OIDC provider

🔹 Step 8 — Configure EC2 Bastion Machine

        Install tools:
        
              sudo yum install -y kubectl
              aws eks update-kubeconfig --region ap-south-1 --name my-cluster


Used for checking:

          Pods
          
          Deployments
          
          Services

🔹 Step 9 — Install Argo CD using Helm

                helm repo add argo https://argoproj.github.io/argo-helm
                helm repo update
                helm install argocd argo/argo-cd -n argocd --create-namespace


            Expose UI via NodePort

🔹 Step 10 — Create Application in Argo CD

Open Argo CD UI

Click NEW APP

          Fill details:
          
                  Field	Value
                  Application Name -	brain-task-app
                  Project -	default
                  Sync Policy-	Automatic
                  Repository URL -	https://github.com/<*********>/<*******>.git
                  Revision -	HEAD
                  Path -	manifest
                  Cluster URL -	https://kubernetes.default.svc
                  Namespace -	default

         Click Create → Sync

🔹 Step 11 — Push All Files to GitHub

              Push:
              
                      Source code
                      
                      Dockerfile
                      
                      buildspec.yaml
                      
                      Kubernetes manifests
              
              CodePipeline will automatically:
              
                      Build → Push to ECR → Trigger ArgoCD sync → Deploy to EKS

🔹 Step 12 — Monitor Logs

              CloudWatch Logs to monitor:
              
              CodeBuild
              
              Pipeline execution
              
              Application logs

🎉 Final Result

Finally a fully automated CI/CD pipeline:

             GitHub → CodePipeline → CodeBuild → ECR → Argo CD → EKS
