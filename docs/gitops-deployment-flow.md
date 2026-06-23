# GitOps Deployment Flow

This document describes how application code becomes a running Kubernetes workload in the JSAPPINF platform.

The platform separates infrastructure provisioning from application delivery.

```text
Terraform creates the platform.
GitLab CI builds application images.
Helm defines Kubernetes application packaging.
ArgoCD deploys workloads from GitOps desired state.
```

---

## Deployment Flow

```text
Developer
  -> JSAPP source repository
  -> GitLab CI pipeline
  -> Docker image build
  -> AWS ECR

Developer
  -> helmchartsappjs repository
  -> Helm charts and values

Developer
  -> jsappinf-gitops repository
  -> ArgoCD Applications

ArgoCD
  -> reads GitOps desired state
  -> pulls Helm chart configuration
  -> deploys workloads to EKS

EKS
  -> runs application namespaces
  -> runs services and pods
  -> exposes applications through ingress-nginx
```

---

## Main Delivery Path

The intended application delivery path is:

```text
Git change
  -> CI pipeline
  -> image build
  -> push to AWS ECR
  -> Helm/GitOps desired state update
  -> ArgoCD sync
  -> Kubernetes rollout
```

Applications are not manually deployed with `kubectl` as the main workflow.

Manual `kubectl` is used mainly for:

```text
debugging
verification
runtime inspection
troubleshooting
```

---

## Repository Responsibilities

### JSAPP

Owns application source code.

```text
Node.js services
Dockerfiles
application routes
health endpoints
database access code
GitLab CI application image build
image versioning
```

### helmchartsappjs

Owns Kubernetes application packaging.

```text
Deployment templates
Service templates
Ingress templates
values.yaml
health probes
service ports
workload scheduling values
stable/burst model
```

### jsappinf-gitops

Owns runtime desired state.

```text
ArgoCD Applications
app-of-apps model
environment application references
GitOps sync model
platform runtime resources
```

### jsappinf-platform

Owns infrastructure and platform foundations.

```text
Terraform infrastructure
VPC
EKS
RDS
ECR
Route 53
KMS
Secrets Manager
platform add-ons
component standards
architecture decisions
```

---

## Current Application Services

```text
ui-service       port 3003
user-service     port 3000
product-service  port 3001
order-service    port 3002
```

---

## Runtime Communication

The application uses Kubernetes service discovery for internal communication.

Example runtime flow:

```text
ui-service
  -> order-service
  -> user-service
  -> product-service
  -> RDS PostgreSQL
```

The services are deployed into Kubernetes and connected through service DNS, environment variables, runtime secrets, and PostgreSQL-backed persistence.

---

## Image Delivery

Application images are built through GitLab CI and pushed to AWS ECR.

The project uses immutable image tagging for safer deployment behavior.

```text
service source change
  -> Git tag / versioned image
  -> GitLab CI build
  -> ECR push
  -> Helm values update
  -> ArgoCD rollout
```

Mutable `latest` image usage is avoided for application deployment stability.

---

## ArgoCD Role

ArgoCD is responsible for reconciling the desired state from Git into the Kubernetes cluster.

ArgoCD manages:

```text
application deployment state
namespace-level application workloads
Helm chart references
sync status
health status
rollout visibility
```

This provides a clear separation between:

```text
build process
artifact storage
deployment intent
runtime reconciliation
```

---

## Why This Model Matters

The GitOps model improves the platform because it provides:

```text
repeatable deployments
versioned runtime desired state
clear audit trail
separation of infrastructure and application delivery
reduced manual Kubernetes changes
better rollback and troubleshooting model
```

---

## Summary

The JSAPPINF deployment model demonstrates a production-like delivery flow:

```text
GitLab CI builds artifacts.
AWS ECR stores images.
Helm describes Kubernetes workloads.
ArgoCD reconciles runtime desired state.
EKS runs the platform and applications.
```
