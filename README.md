# ih-project-2

## Overview

This is a multi-service voting application that leverages a CI/CD workflow with GitHub Actions, Docker, and Kubernetes on AWS EKS. The application consists of three main components: `worker`, `vote`, and `result`, each built and deployed as a Docker container.

## Directory Structure

```
.
├── worker/            # Source & Dockerfile for 'worker' service
├── vote/              # Source & Dockerfile for 'vote' service
├── result/            # Source & Dockerfile for 'result' service
├── K8s/
│   ├── pvc/           # PersistentVolumeClaim manifests
│   ├── deployment/    # Deployment manifests
│   ├── services/      # Service manifests
│   └── ingress/       # Ingress manifests
└── .github/
    └── workflows/
        └── cicd.yml  # GitHub Actions workflow file
```

---

## CI/CD Pipeline (`.github/workflows/cicd.yml`)

### Triggers

- On any push to `main`
- On any pull request targeting `main`

### Pipeline Steps

#### 1. Build and Push Docker Images

- Checks out the repository
- Logs into Docker Hub using GitHub secrets
- Builds Docker images for each service:
  - `worker`
  - `vote`
  - `result`
- Tags images as `${DOCKERHUB_USERNAME}/<service>:latest`
- Pushes the images to Docker Hub

#### 2. Deploy to AWS EKS

- Configures AWS credentials using secrets
- Installs `eksctl` for EKS management
- Updates kubeconfig using specified cluster name and region (**NOTE:** these are pulled from secrets)
- Applies Kubernetes manifests in the following order:
  1. PersistentVolumeClaims (`K8s/pvc/`)
  2. Deployments (`K8s/deployment/`)
  3. Services (`K8s/services/`)
  4. Ingresses (`K8s/ingress/`)

---

## Required GitHub Secrets

Add these *repository-level secrets* in **Settings → Secrets and variables → Actions**:

| Secret Name            | Used For                 | Description                                         |
|------------------------|--------------------------|-----------------------------------------------------|
| DOCKERHUB_USERNAME     | Docker login & tagging   | Docker Hub account username                         |
| DOCKERHUB_TOKEN        | Docker login             | Docker Hub password or access token                 |
| AWS_ACCESS_KEY_ID      | AWS credentials          | AWS IAM Access Key ID for deploy permissions        |
| AWS_SECRET_ACCESS_KEY  | AWS credentials          | AWS IAM Secret Access Key                           |
| AWS_CLUSTERNAME        | EKS cluster connection   | The **name** of your AWS EKS cluster                |
| AWS_REGION             | EKS cluster connection   | AWS region where your EKS cluster is located        |
| NAMESPACE              | EKS cluster configuration| Creates and and switches Namespace                  |

**All secrets are required for your pipeline to operate correctly!**

---

## Usage

1. **Push changes to `main`**  
   or **Open a PR targeting `main`**  
   → This will trigger the workflow.

2. The workflow automatically:
   - Builds and pushes Docker images
   - Updates images on your EKS cluster by applying manifests in the correct order

3. **Kubernetes Manifests**
   - Make sure to reference your fresh images in deployment manifests:
     ```yaml
     image: ${{ secrets.DOCKERHUB_USERNAME }}/worker:latest
     image: ${{ secrets.DOCKERHUB_USERNAME }}/vote:latest
     image: ${{ secrets.DOCKERHUB_USERNAME }}/result:latest
     ```
   - Place each manifest in the appropriate subdirectory under `K8s/` as per the workflow.

---

## Example: Updating Your Application

To roll out new code:
- Update your application code or Dockerfile in any service subfolder.
- Commit, push to `main` (or create a PR).
- The workflow will:
  - Build and push new images to Docker Hub
  - Apply the latest (`K8s/`) manifests to your EKS cluster, triggering rolling updates.

---

## Troubleshooting

- **Pipeline Fails on Docker:**  
  Check that your Docker Hub credentials (`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`) are valid.
- **AWS or K8s Step Fails:**  
  Ensure your AWS secrets and cluster info (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_CLUSTERNAME`, `AWS_REGION`) are correct and your IAM user has sufficient permissions.
- **Deployment Issues:**  
  - Confirm manifests in `K8s/` are valid and reference the **latest image tags**.
  - Review the deployment order to ensure PVCs, Deployments, Services, and Ingress are in the correct structure/subfolders.

