# Trendify App: Production Deployment Repository

This repository contains the production-ready build files for the Trendify application. It is intentionally structured for DevOps practice and deployment exercises, so learners can focus on CI/CD pipelines, hosting, containerization, and infrastructure setup rather than application development.

This repository represents the production-build and deployment stage of the application lifecycle. It is not the application source repository.

## 🌐 Trendify Project Ecosystem

This repository is part of the **Trendify Enterprise Cloud Platform**, a fully automated, GitOps-driven, two-tier application stack. To enforce a strict separation of concerns, the architecture is decoupled into four distinct repositories:

1. **[Trendify-Platform](https://github.com/shgupta140-max/Trendify-Platform.git) (Automation & Observability):**
   * **Role:** The foundational layer. Contains Terraform code to provision the Jenkins CI/CD automation server and Helm configurations to deploy the centralized monitoring stack (`kube-prometheus-stack` & `blackbox-exporter`).

2. **[Trendify-Infra]( https://github.com/shgupta140-max/Trendify-Infra.git) (Cloud Infrastructure):**
   * **Role:** The immutable AWS infrastructure layer. Contains Terraform modules to provision the production-grade Amazon EKS cluster (`trendstore-cluster` in `ap-south-1`), VPC networks, IAM Access Entries, and the AWS ALB Controller.

3. **[Trendify-App](https://github.com/shgupta140-max/Trendify-App.git) (Application Code & CI):**
   * **Role:** The product layer. Houses the Node.js application source code, Dockerfile, and the Continuous Integration (CI) Jenkins pipeline. 
   * **Connection:** This pipeline builds the image, pushes it to DockerHub, and automatically commits the new image tag directly into the `Trendify-GitOps` repository.

4. **[Trendify-GitOps](https://github.com/shgupta140-max/Trendify-GitOps.git) (Cluster State & CD):**
   * **Role:** The single source of truth for the Kubernetes cluster state. Contains the application deployment manifests and Kustomize overlays.
   * **Connection:** Triggered by commits from `Trendify-App`, this Jenkins pipeline requires manual Slack approval before deploying changes to the `Trendify-Infra` EKS cluster and dynamically injecting the AWS ALB URL into the monitoring probes.

---

## Repository Connections

| Component | Location | Responsibility |
| --- | --- | --- |
| Application deployment repository | [Trendify-App](https://github.com/shgupta140-max/Trendify-App) | Stores the compiled `dist` files, Dockerfile, Nginx configuration, and Jenkins pipeline. |
| Container image | [Docker Hub: shgupta140/trendify-app](https://hub.docker.com/r/shgupta140/trendify-app) | Stores versioned images and the `latest` image for deployment. |
| GitOps repository | [Trendify-GitOps](https://github.com/shgupta140-max/Trendify-GitOps) | Stores Kubernetes manifests. Jenkins updates `kubedefs/01-deployment.yml` with the newly built image tag. |

### Delivery flow

1. Jenkins builds a Docker image from this repository.
2. The image is tagged with the Jenkins build number and with `latest`.
3. Jenkins pushes both tags to Docker Hub.
4. Jenkins updates the image tag in the [Trendify-GitOps](https://github.com/shgupta140-max/Trendify-GitOps) deployment manifest and pushes the change to its `main` branch.
5. The GitOps deployment process can then apply the updated Kubernetes manifest to the target cluster.

The Jenkins pipeline uses the following image format:

```text
docker.io/shgupta140/trendify-app:v<BUILD_NUMBER>
```

## What This Repository Contains

- `dist/`: compiled and production-ready static files, including HTML, CSS, JavaScript, and assets such as images and fonts.
- `Dockerfile`: packages the `dist` directory into an Nginx container.
- `nginx.conf`: serves the application on port `3000`, supports frontend routing, and caches static assets.
- `Jenkinsfile`: builds and pushes the Docker image, then updates the GitOps repository.
- `.dockerignore`: controls which files are excluded from the Docker build context.

The `dist` files are ready to deploy to:

- Web servers such as Nginx or Apache
- Cloud storage platforms such as AWS S3, Azure Blob Storage, or Google Cloud Storage
- Containerized environments using Docker and Nginx
- Kubernetes clusters
- CI/CD pipeline demonstrations

## Local Container Usage

Build and run the image from the repository root:

```bash
docker build -t trendify-app:local .
docker run --rm -p 3000:3000 trendify-app:local
```

Open <http://localhost:3000> in a browser. Nginx listens on port `3000` inside the container and serves the files from `/usr/share/nginx/html`.

## Purpose

This repository is designed for:

- DevOps beginners
- CI/CD practice
- Deployment pipeline testing
- Docker and Kubernetes deployment exercises
- Web server configuration practice
- Reverse proxy and load balancer setup

The goal is to simulate real-world deployment scenarios using already built application files.

## Why There Is No `package.json`

This repository intentionally does not include:

- `package.json`
- `node_modules`
- Source code such as `src/`
- Build tool configuration

It contains only the final production build output, not the development source code. In a typical application workflow, developers write source code and use tools such as Node.js, Webpack, Vite, React, or another framework to generate the `dist` directory. Only that compiled output is deployed to servers.

Since this repository contains already compiled output:

- No application dependencies are required.
- No application build process is required.
- No `package.json` is needed.

## Deployment Notes

This section describes the recommended order for deploying the complete Trendify platform. The repositories are intentionally deployed in dependency order: platform automation first, AWS infrastructure second, application manifests third, and monitoring last.

### Prerequisites

Install and configure the following tools on the workstation or deployment host:

- Git
- Terraform
- AWS CLI
- Docker
- `kubectl`
- Helm, if Helm commands are required by the platform or GitOps pipelines

You also need an AWS account with permission to create the resources defined by the Terraform modules, a GitHub account or token with access to all four repositories, and Docker Hub credentials for `shgupta140/trendify-app`.

### 1. Clone all project repositories

Clone the four repositories into the same parent directory. Keeping them together makes it easier to inspect the Terraform, Jenkins, application, and GitOps configuration as one deployment:

```bash
git clone https://github.com/shgupta140-max/Trendify-Platform.git
git clone https://github.com/shgupta140-max/Trendify-Infra.git
git clone https://github.com/shgupta140-max/Trendify-App.git
git clone https://github.com/shgupta140-max/Trendify-GitOps.git
```

The expected repository responsibilities are:

- `Trendify-Platform`: Jenkins infrastructure and the monitoring stack configuration.
- `Trendify-Infra`: AWS VPC, EKS, IAM, and AWS Load Balancer Controller infrastructure.
- `Trendify-App`: application image build and Docker Hub publishing.
- `Trendify-GitOps`: Kubernetes manifests and GitOps deployment configuration.

### 2. Configure AWS and deploy the platform layer

1. Configure the AWS CLI with credentials for the target account:

   ```bash
   aws configure
   aws sts get-caller-identity
   ```

   Confirm that the returned account and region are the intended deployment target. The infrastructure documentation currently identifies `ap-south-1` as the target region.

2. Change into the platform repository and review its Terraform variables:

   ```bash
   cd trendify-platform
   # Edit variable.tf and provide every required environment-specific value.
   terraform init
   terraform validate
   terraform plan
   ```

   Set values such as the AWS region, networking parameters, instance settings, domain or access settings, and any credentials or IDs required by the repository. Do not commit AWS credentials, tokens, or other secrets to Git.

3. After reviewing the plan, create the Jenkins server and platform resources:

   ```bash
   terraform apply
   ```

   Save the Jenkins server address and any outputs required to connect to it. Wait until the server is reachable before creating pipeline jobs.

### 3. Configure Jenkins pipeline jobs

Log in to the Jenkins server created by the platform Terraform configuration. Create four Pipeline jobs, each configured to load its `Jenkinsfile` from the corresponding repository and branch:

| ----- Jenkins job  --------- | ---- Repository ------- | ---- Purpose ------------------------------------------------------------------------------------------|
| `Trendify-App-Pipeline`      | `Trendify-App.git`      | Builds the application image, pushes it to Docker Hub, and updates the image tag in `Trendify-GitOps`. |
| `Trendify-GitOps-Pipeline`   | `Trendify-GitOps.git`   | Applies the application Kubernetes manifests to the EKS cluster.                                       |
| `Trendify-Infra-Pipeline`    | `Trendify-Infra.git`    | Creates or updates the AWS infrastructure and EKS cluster.                                             |
| `Trendify-Platform-Pipeline` | `trendify-platform.git` | Deploys the `kube-prometheus-stack` and related monitoring configuration.                              |

Before running the jobs, configure the required Jenkins credentials and permissions:

- GitHub credentials for checking out and pushing to the repositories.
- Docker Hub credentials for publishing `shgupta140/trendify-app`.
- AWS credentials or an IAM role usable by the Jenkins agent.
- Cluster access for `kubectl` and Helm after EKS is created.
- Slack credentials or webhook configuration if the GitOps pipeline uses manual Slack approval.

The application pipeline expects the GitOps manifest at `kubedefs/01-deployment.yml` and updates the `main` branch with the new image tag.

### 4. Create the EKS cluster

After the Jenkins jobs exist, run the infrastructure pipeline or apply the infrastructure Terraform directly from the `Trendify-Infra` repository:

```bash
cd ../Trendify-Infra
terraform init
terraform validate
terraform plan
terraform apply
```

Wait for the EKS cluster, networking, IAM access, and AWS Load Balancer Controller resources to become ready. Configure `kubectl` for the cluster using the command and outputs provided by the infrastructure repository, for example:

```bash
aws eks update-kubeconfig --region ap-south-1 --name trendstore-cluster
kubectl get nodes
```

Run the `Trendify-Infra-Pipeline` instead when Jenkins is the approved execution path. Do not continue until the cluster is reachable from the Jenkins agent.

### 5. Deploy the application through GitOps

Once the EKS cluster is ready, trigger `Trendify-GitOps-Pipeline`. The pipeline should apply the Kubernetes objects and Kustomize configuration from `Trendify-GitOps` to the cluster. Complete any configured Slack approval before the deployment proceeds.

Verify that the application resources are running:

```bash
kubectl get deployments,pods,services,ingresses -A
```

If the application image has not yet been published, run `Trendify-App-Pipeline` first. That pipeline builds the Docker image, pushes the versioned and `latest` tags to Docker Hub, and commits the new versioned image tag to `Trendify-GitOps`. Then rerun `Trendify-GitOps-Pipeline` so the cluster receives that image.

### 6. Deploy monitoring

After the application and its ingress are available, trigger `Trendify-Platform-Pipeline`. This pipeline deploys the `kube-prometheus-stack` and `blackbox-exporter` configuration from `trendify-platform` and configures monitoring for the cluster and application endpoint.

Verify the monitoring components:

```bash
kubectl get pods -A
helm list -A
```

The final deployment order is therefore:

```text
trendify-platform Terraform
        -> Jenkins server and pipeline jobs
        -> Trendify-Infra Terraform / EKS cluster
        -> Trendify-App-Pipeline / Docker Hub image
        -> Trendify-GitOps-Pipeline / application Kubernetes objects
        -> Trendify-Platform-Pipeline / monitoring stack
```

Kubernetes deployment details, services, cluster configuration, and Kustomize overlays belong in the [Trendify-GitOps](https://github.com/shgupta140-max/Trendify-GitOps) repository. The exact Terraform variables, Jenkins parameters, and secret names must follow the configuration in each connected repository.
