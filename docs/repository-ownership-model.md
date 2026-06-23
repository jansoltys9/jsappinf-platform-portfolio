# Repository Ownership Model

This document explains how the JSAPPINF project is split across repositories and why this separation matters.

The goal is to avoid a single monolithic repository and make responsibilities clear across infrastructure, application code, Kubernetes packaging, and runtime deployment state.

---

## Repository Overview

```text
terraform-modules
  -> reusable Terraform modules

jsappinf-platform
  -> environment infrastructure wiring and platform standards

JSAPP
  -> application source code and CI build logic

helmchartsappjs
  -> Helm charts and Kubernetes application packaging

jsappinf-gitops
  -> ArgoCD Applications and runtime desired state
```

---

## Ownership Diagram

```text
terraform-modules
  -> jsappinf-platform
  -> AWS infrastructure

JSAPP
  -> GitLab CI
  -> AWS ECR

helmchartsappjs
  -> Helm chart definitions
  -> ArgoCD

jsappinf-gitops
  -> ArgoCD Applications
  -> EKS runtime desired state

ArgoCD
  -> EKS cluster
  -> running platform and application workloads
```

---

## terraform-modules

This repository owns reusable Terraform modules.

Examples:

```text
VPC
EKS
RDS
future reusable modules
```

The purpose of this repository is to provide reusable building blocks.

It should not contain environment-specific runtime logic, application deployment decisions, or GitOps application state.

Good examples of responsibility:

```text
networking module inputs and outputs
EKS module contract
RDS module contract
module versioning
module compatibility notes
```

---

## jsappinf-platform

This is the main infrastructure and platform repository.

It owns environment wiring and infrastructure composition.

Examples:

```text
dev environment
Terraform backend usage
VPC/EKS/RDS module wiring
ECR repositories
platform add-ons
KMS responsibility model
External Secrets integration
cert-manager integration
ingress-nginx installation
Karpenter integration
component registry
architecture decisions
checkpoint documentation
```

This repository is responsible for composing reusable modules into an environment.

It also defines platform standards such as:

```text
component ownership
Terraform vs GitOps boundaries
KMS key responsibility
Secrets Manager and ESO flow
addon ownership model
cost-aware dev behavior
```

---

## JSAPP

This repository owns the application source code.

Examples:

```text
Node.js services
Express routes
application logic
database access code
Dockerfiles
health endpoints
GitLab CI build pipeline
service image tags
```

The JSAPP repository produces container images.

It does not own AWS infrastructure or Kubernetes runtime desired state.

Expected application image flow:

```text
source code change
  -> GitLab CI build
  -> Docker image
  -> AWS ECR
  -> versioned image tag
```

---

## helmchartsappjs

This repository owns Kubernetes application packaging.

Examples:

```text
Deployment templates
Service templates
Ingress templates
values.yaml
container ports
service ports
health probes
environment variables
secret references
workload scheduling
stable/burst deployment model
```

This repository defines how the application should run on Kubernetes.

It does not build application images and does not provision AWS infrastructure.

---

## jsappinf-gitops

This repository owns runtime desired state for ArgoCD.

Examples:

```text
ArgoCD Applications
app-of-apps structure
environment application references
platform runtime resources
sync policies
GitOps deployment flow
```

This repository tells ArgoCD what should be deployed.

It should avoid storing raw secret values.

GitOps manifests should reference secret delivery mechanisms such as External Secrets Operator instead of embedding sensitive values.

---

## Responsibility Boundaries

```text
Terraform:
  creates infrastructure and platform foundations

GitLab CI:
  builds and publishes application artifacts

AWS ECR:
  stores container images

Helm:
  describes Kubernetes application shape

ArgoCD:
  reconciles desired runtime state into EKS

External Secrets Operator:
  injects runtime secrets from AWS Secrets Manager

Application code:
  implements business behavior
```

---

## Why This Split Matters

This model is more production-like because it separates:

```text
infrastructure lifecycle
application build lifecycle
Kubernetes packaging lifecycle
runtime deployment lifecycle
secret delivery lifecycle
platform standards and architecture decisions
```

It also improves:

```text
auditability
change ownership
rollback clarity
security boundaries
team responsibility separation
future multi-environment support
```

---

## Example Change Flow

### Application code change

```text
JSAPP change
  -> GitLab CI image build
  -> ECR image push
  -> Helm/GitOps image reference update
  -> ArgoCD sync
  -> Kubernetes rollout
```

### Infrastructure change

```text
jsappinf-platform change
  -> Terraform plan
  -> Terraform apply
  -> AWS/EKS infrastructure update
```

### Helm chart change

```text
helmchartsappjs change
  -> chart template/value update
  -> ArgoCD detects desired state change
  -> Kubernetes workload update
```

### Runtime desired state change

```text
jsappinf-gitops change
  -> ArgoCD Application update
  -> sync and health validation
```

---

## Portfolio Summary

The repository ownership model shows that JSAPPINF is not only a running EKS lab, but also a structured platform engineering project.

The key design idea is:

```text
keep infrastructure, application code, Kubernetes packaging, and runtime desired state separate
```

This creates a cleaner path toward multi-environment support, safer deployments, and more realistic platform ownership.
