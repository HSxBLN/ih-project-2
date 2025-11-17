# ih-project-2

## Overview

This repository implements a multi-component voting application, featuring:
- `worker`, `vote`, and `result` services (each likely with its own Dockerfile).
- Automated build, push, and deployment orchestrated via GitHub Actions and Kubernetes on AWS EKS.

---

## CI/CD Pipeline

The workflow in `.github/workflows/cicd.yml` provides:

### Trigger Conditions

- On any push or pull request to the `main` branch.

### Pipeline Stages

#### 1. Build and Push Docker Images

- Checks out code.
- Logs into Docker Hub using provided secrets.
- Builds Docker images for:  
  - `worker` (from `./worker/Dockerfile`)
  - `vote` (from `./vote/Dockerfile`)
  - `result` (from `./result/Dockerfile`)
- Tags each image as `${DOCKERHUB_USERNAME}/[service]:latest`.
- Pushes these images to Docker Hub.

#### 2. Deploy to EKS

- Configures AWS credentials.
- Installs `eksctl`.
- Updates kubeconfig for the cluster `romain-voting-app-cluster` in the `eu-west-3` region.
- Applies Kubernetes manifests found in the `K8s/` directory.

---

## Requirements

### Secrets

These secrets must be added in your repository settings (Settings → Secrets and variables → Actions):

| Secret Name                      | Used For                  | Description                                              |
|-----------------------------------|---------------------------|----------------------------------------------------------|
| `DOCKERHUB_USERNAME`              | Docker login, image tags  | Your Docker Hub username                                 |
| `DOCKERHUB_TOKEN`                 | Docker login              | Personal access token/password for Docker Hub            |
| `AWS_ACCESS_KEY_ID`               | AWS credentials           | AWS access key ID to authenticate GitHub Actions         |
| `AWS_SECRET_ACCESS_KEY`           | AWS credentials           | AWS secret access key for above access key               |

*All above secrets are **required** for the pipeline to successfully run.*

---

## Setup and Usage

### Prerequisites

- AWS EKS cluster named `romain-voting-app-cluster` in region `eu-west-3`, already running.
- Docker Hub account with permissions for above username.
- The following directory structure:
  ```
  .
  ├── worker/
  │   └── Dockerfile
  ├── vote/
  │   └── Dockerfile
  ├── result/
  │   └── Dockerfile
  └── K8s/
      ├── deployment.yaml
      ├── service.yaml
      └── ... (any other manifests)
  ```

### Quick Start

1. Push to the `main` branch or open a pull request targeting `main`.
2. GitHub Actions will:
   - Build and push Docker images.
   - Apply all manifests in the `K8s/` directory to your EKS cluster.
3. (Optional) Monitor Action runs under the "Actions" tab for this repo.

---

## K8s Manifests

All manifests under `K8s/` are automatically applied to the cluster after every successful build.

- Ensure manifests reference images using `${DOCKERHUB_USERNAME}/worker:latest`, etc.
- Example snip in a deployment yaml:
  ```yaml
  containers:
    - name: worker
      image: your_dockerhub_username/worker:latest
      ...
  ```

---

## Customization

- To change cluster/region, update the corresponding parts in the workflow YAML.
- To add environments or secrets (e.g., database credentials), extend the workflow and secrets in GitHub as needed.

---

## Troubleshooting

- Make sure all required secrets are set up _before_ triggering workflows.
- Check Action run logs for details on build, push, and deployment stages.
- Validate the cluster name and region match your AWS setup.

---

## License

[Specify your license here]
