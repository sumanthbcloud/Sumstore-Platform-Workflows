# Sumstore-Platform-Workflows — Centralized Reusable CI Pipelines

This repository provides standardized, audited GitHub Actions reusable workflows (`workflow_call`) consumed by all microservices in the SUM Store platform.

---

## Architecture & Golden Path Pipeline

Each service workflow enforces the SUM Store Golden Path:
1. **Source Checkout**: Shallow fetch with full commit history for version tagging.
2. **Security & Vulnerability Scanning**: Aqua Trivy container image scanning with non-blocking audit thresholds.
3. **Container Build**: Multi-stage Docker builds using buildx and layer caching.
4. **AWS Authentication**: Passwordless OpenID Connect (OIDC) federation assuming the central deployment role (`github-actions-sumstore-apps`).
5. **Amazon ECR Publishing**: Pushing immutable container images tagged with the exact Git commit SHA.
6. **GitOps Synchronization**: Automatic promotion of the new image tag into `Sumstore-GitOps` (`apps/<service>/values.yaml`).

---

## Reusable Workflows

| Workflow | Technology Stack | Consumer Workload(s) |
| :--- | :--- | :--- |
| [`.github/workflows/node-service.yml`](.github/workflows/node-service.yml) | Node.js (Express) | `cart`, `catalogue`, `user` |
| [`.github/workflows/python-service.yml`](.github/workflows/python-service.yml) | Python (Flask) | `payment` |
| [`.github/workflows/go-service.yml`](.github/workflows/go-service.yml) | Golang | `dispatch` |
| [`.github/workflows/java-service.yml`](.github/workflows/java-service.yml) | Java (Spring Boot) | `shipping` |
| [`.github/workflows/php-service.yml`](.github/workflows/php-service.yml) | PHP (Symfony) | `ratings` |
| [`.github/workflows/web-service.yml`](.github/workflows/web-service.yml) | Nginx & Static Assets | `web` |

---

## Standard Inputs & Secrets Contract

### Inputs
- `service_name`: Identifier of the microservice (e.g. `cart`).
- `build_context`: Path to the Docker build context (e.g. `./app/cart`).
- `dockerfile`: Path to the service Dockerfile (e.g. `./app/cart/Dockerfile`).
- `gitops_values_path`: Relative path in `Sumstore-GitOps` to update (e.g. `apps/cart/values.yaml`).

### Secrets & IAM Configuration
- **AWS OIDC Role**: `arn:aws:iam::961544628309:role/github-actions-sumstore-apps`
- **GitOps Token**: `GITOPS_TOKEN` (Fine-grained Personal Access Token with repository contents write access).
- **Optional Integrations**: SonarQube, SCA, and Slack blocks are reserved for planned security passes.
