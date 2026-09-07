# Trendify App: Production Deployment Repository

This repository contains the production-ready build files for the Trendify application. It is intentionally structured for DevOps practice and deployment exercises, so learners can focus on CI/CD pipelines, hosting, containerization, and infrastructure setup rather than application development.

This repository represents the production-build and deployment stage of the application lifecycle. It is not the application source repository.

## 🌐 Trendify Project Ecosystem

This repository is part of the **Trendify Enterprise Cloud Platform**, a fully automated, GitOps-driven, two-tier application stack. To enforce a strict separation of concerns, the architecture is decoupled into four distinct repositories:

1. **[Trendify-Platform](https://github.com/shgupta140-max/trendify-platform.git) (Automation & Observability):**
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

- The Jenkins pipeline requires Docker access and configured Jenkins credentials for Docker Hub and GitHub.
- The pipeline expects the GitOps manifest at `kubedefs/01-deployment.yml`.
- The GitOps repository currently uses the `main` branch.
- Kubernetes deployment details, services, and cluster configuration belong in the [Trendify-GitOps](https://github.com/shgupta140-max/Trendify-GitOps) repository.
