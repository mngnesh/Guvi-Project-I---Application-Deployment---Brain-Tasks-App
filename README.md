# Brain Tasks App — AWS DevOps Deployment

## 1. Project Overview

This project deploys the Brain Tasks React application to a production-style AWS environment using Docker, Amazon ECR, Amazon EKS, AWS CodeBuild, AWS CodePipeline, Kubernetes, GitHub, and CloudWatch.

### Application

- Application: Brain Tasks App
- Application port: `3000`
- GitHub repository: https://github.com/mngnesh/Guvi-Project-I---Application-Deployment---Brain-Tasks-App
- AWS Region: `ap-south-1` (Asia Pacific - Mumbai)

## 2. Architecture

```text
GitHub
   |
   v
AWS CodePipeline
   |
   +---- Source
   |
   v
AWS CodeBuild
   |
   +---- Docker build
   |
   +---- Push image to Amazon ECR
   |
   v
Amazon ECR
   |
   v
Amazon EKS
   |
   +---- Kubernetes Deployment
   |
   +---- Kubernetes Service (LoadBalancer)
   |
   v
Brain Tasks React Application
```

## 3. AWS Resources

| Resource | Value |
|---|---|
| AWS Region | `ap-south-1` |
| EKS Cluster | `brain-tasks-cluster` |
| ECR Repository | `brain-tasks-app` |
| CodeBuild Project | `brain-tasks-codebuild` |
| CodePipeline | `brain-tasks-pipeline` |
| Kubernetes Deployment | `brain-tasks-app` |
| Kubernetes Service | `brain-tasks-service` |
| Container Port | `3000` |

## 4. Live Application

Application URL:

http://a21be09a0fe1442df9a375faa8d1344f-1465668506.ap-south-1.elb.amazonaws.com:3000

The application is exposed through a Kubernetes `LoadBalancer` Service.

### Load Balancer details

- Load Balancer type: Network Load Balancer (NLB)
- Load Balancer ARN: `arn:aws:elasticloadbalancing:ap-south-1:391505362986:loadbalancer/net/a387d9d30d7404230a324ee54239b259/d8f26a5073e9c540`
- DNS name: `a387d9d30d7404230a324ee54239b259-d8f26a5073e9c540.elb.ap-south-1.amazonaws.com`
- Scheme: `internet-facing`
- Port: `3000`

## 5. Repository Structure

```text
Brain-Tasks-App/
|
+-- dist/
+-- k8s/
|   +-- deployment.yaml
|   +-- service.yaml
|
+-- Dockerfile
+-- nginx.conf
+-- buildspec.yml
+-- README.md
```

## 6. Dockerization

The application is packaged into a Docker image using Nginx to serve the built React application.

### Build the image

```bash
docker build -t brain-tasks-app:1.0 .
```

### Run locally

```bash
docker run -d --name brain-tasks-app-container -p 3000:3000 brain-tasks-app:1.0
```

The application can then be tested at:

```text
http://localhost:3000
```

### Check the container

```bash
docker ps
```

## 7. Amazon ECR

An Amazon ECR repository named `brain-tasks-app` was created in `ap-south-1`.

Example ECR repository URI:

```text
391505362986.dkr.ecr.ap-south-1.amazonaws.com/brain-tasks-app
```

Docker images are built by CodeBuild and pushed to this repository.

Images are tagged using the Git commit/source version so that each pipeline build can produce a unique image tag.

## 8. Kubernetes / Amazon EKS

### EKS Cluster

```text
brain-tasks-cluster
```

The EKS cluster is running in:

```text
ap-south-1
```

### Kubernetes Deployment

The application is deployed using:

```text
k8s/deployment.yaml
```

The deployment uses the dynamically supplied `IMAGE_URI` value from CodeBuild/CodePipeline.

### Kubernetes Service

The application is exposed using:

```text
k8s/service.yaml
```

The Service type is:

```yaml
type: LoadBalancer
```

The application listens on port `3000`.

### Useful commands

Check the cluster:

```bash
kubectl get nodes
```

Check pods:

```bash
kubectl get pods
```

Check deployment:

```bash
kubectl get deployments
```

Check service:

```bash
kubectl get svc
```

Check endpoints:

```bash
kubectl get endpoints
```

## 9. AWS CodeBuild

CodeBuild project:

```text
brain-tasks-codebuild
```

The project uses `buildspec.yml` to automate the Docker build and ECR push.

### Build process

The build performs these operations:

1. Authenticate to Amazon ECR.
2. Generate an image tag from the source commit.
3. Build the Docker image.
4. Push the image to ECR.
5. Export `IMAGE_URI` and `IMAGE_TAG`.
6. Provide the Kubernetes manifests as pipeline artifacts.

## 10. buildspec.yml

The buildspec contains the following major stages:

```text
PRE_BUILD
    |
    +-- Login to ECR
    +-- Generate image tag
    +-- Generate IMAGE_URI

BUILD
    |
    +-- Build Docker image

POST_BUILD
    |
    +-- Push Docker image to ECR
    +-- Export IMAGE_URI
    +-- Export IMAGE_TAG
    +-- Upload Kubernetes manifests
```

## 11. AWS CodePipeline

Pipeline:

```text
brain-tasks-pipeline
```

The pipeline contains three stages:

```text
GitHub Source
      |
      v
AWS CodeBuild
      |
      v
Amazon EKS Deploy
```

### Source

GitHub is used as the source provider.

The pipeline tracks the `main` branch.

### Build

AWS CodeBuild project:

```text
brain-tasks-codebuild
```

The CodeBuild project:

- Builds the Docker image.
- Pushes the image to ECR.
- Exports `IMAGE_URI`.

### Deploy

The deployment action uses:

```text
Amazon EKS
Kubectl
```

The following Kubernetes manifests are deployed:

```text
k8s/deployment.yaml
k8s/service.yaml
```

The dynamically generated ECR image URI is passed into the Kubernetes deployment as:

```text
IMAGE_URI
```

This allows every pipeline execution to deploy the image produced by the corresponding build.

## 12. CloudWatch Monitoring

CloudWatch Logs are enabled for the CI/CD workflow.

### CodeBuild logs

```text
/aws/codebuild/brain-tasks-codebuild
```

These logs provide visibility into:

- Source download
- ECR authentication
- Docker image build
- ECR image push
- Build phases
- Build success/failure

### CodePipeline logs

```text
/aws/codepipeline/brain-tasks-pipeline
```

These logs provide visibility into pipeline execution.

## 13. Deployment Verification

The deployment was verified using Kubernetes commands.

Example:

```bash
kubectl get pods
```

The application pod reached:

```text
1/1 Running
```

The Kubernetes Service was verified as:

```text
LoadBalancer
```

The Service endpoint was verified as:

```text
192.168.26.20:3000
```

The application was also tested from inside the Kubernetes cluster and returned the React application's HTML successfully.

The public LoadBalancer URL was tested successfully:

```text
http://a21be09a0fe1442df9a375faa8d1344f-1465668506.ap-south-1.elb.amazonaws.com:3000
```

## 14. CI/CD Verification

The final CodePipeline execution completed successfully:

```text
Source  - Succeeded
Build   - Succeeded
Deploy  - Succeeded
```

This confirms the complete automated deployment flow:

```text
GitHub Push
    |
    v
CodePipeline Source
    |
    v
CodeBuild
    |
    +--> Docker Build
    |
    +--> ECR Push
    |
    v
EKS Deploy
    |
    v
Kubernetes
    |
    v
Live Application
```

## 15. Screenshots

Project screenshots to a `screenshots/` directory and listed here.

Recommended screenshots:

1. GitHub repository
2. Docker image/container running locally
3. Amazon ECR repository and image
4. EKS cluster showing active/running status
5. Kubernetes pods showing `Running`
6. Kubernetes Service showing `LoadBalancer`
7. Live Brain Tasks application
8. CodeBuild successful execution
9. CodePipeline showing Source, Build, and Deploy all successful
10. CloudWatch CodeBuild logs
11. CloudWatch CodePipeline logs


## 16. Conclusion

The Brain Tasks React application was containerized with Docker, stored in Amazon ECR, deployed to Amazon EKS using Kubernetes manifests, and automated through AWS CodePipeline and CodeBuild.

CloudWatch Logs provide monitoring visibility for the CI/CD workflow, and the application is accessible through the Kubernetes LoadBalancer on port `3000`.
