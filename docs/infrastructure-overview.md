# Infrastructure Overview

JSAPPINF is a production-like DevOps platform lab built on AWS EKS.

The platform is designed to demonstrate infrastructure provisioning, Kubernetes platform setup, GitOps deployment, secure secret delivery, ingress, TLS, database integration, and operational troubleshooting.

As of 2026-09-17, DEV runtime is intentionally OFF for cost control. Runtime descriptions below refer to source configuration or previously validated capabilities, not currently running workloads.

---

## High-Level Architecture

```text
Developer / Operator
  -> Git repositories
  -> Terraform infrastructure code
  -> AWS account
  -> VPC
  -> EKS
  -> Platform add-ons
  -> ArgoCD
  -> Helm applications
  -> JSAPP microservices
  -> RDS PostgreSQL
```

---

## AWS Infrastructure Layer

The infrastructure layer includes:

```text
VPC
public subnets
private subnets
routing
NAT gateway
EKS cluster
managed node groups
ECR repositories
RDS PostgreSQL
AWS Secrets Manager
Route 53 DNS
KMS responsibility model
```

Terraform is responsible for provisioning and wiring these resources.

---

## Kubernetes Platform Layer

The EKS cluster hosts both platform components and application workloads.

Main platform components:

```text
ArgoCD
External Secrets Operator
cert-manager
AWS Load Balancer Controller and Gateway API
Prometheus / Grafana
optional Karpenter
```

Each component has a defined ownership boundary. Terraform owns infrastructure and platform foundations, while GitOps owns runtime desired state where appropriate.

---

## Application Layer

The JSAPP application consists of multiple Node.js microservices:

```text
ui-service
user-service
product-service
order-service
```

The previously validated runtime flow is:

```text
Browser / UI
  -> ui-service
  -> order-service
  -> user-service
  -> product-service
  -> RDS PostgreSQL
```

This validates Kubernetes deployment, ingress routing, service discovery, secrets injection, database access, and application-level integration.

---

## Platform Add-ons

### ArgoCD

ArgoCD provides GitOps-based reconciliation of runtime desired state into Kubernetes.

### External Secrets Operator

External Secrets Operator synchronizes secrets from AWS Secrets Manager into Kubernetes Secrets.

### cert-manager

cert-manager automates TLS certificate management.

### ALB and Gateway API

The Terraform-managed AWS Load Balancer Controller reconciles GitOps Gateway and HTTPRoute resources into ALB routing to application Services.

### Karpenter

Karpenter is treated as optional in dev. It can be enabled for capacity validation, but remains disabled or idle by default to control cost.

---

## Cost-Aware Dev Model

The dev environment is designed to be rebuilt and destroyed frequently.

Default dev behavior:

```text
lower baseline capacity
daily rebuild/destroy workflow
optional autoscaling disabled or idle
SPOT burst disabled by default
```

This keeps the environment practical while still demonstrating production-like platform design.

---

## Edge Security Direction

The edge architecture implemented in source and previously validated is:

```text
Internet
  -> CloudFront
  -> AWS WAF Web ACL
  -> Application Load Balancer (configured through Gateway API / HTTPRoute)
  -> JSAPP services
```

Current status:

```text
CloudFront and AWS WAF infrastructure implemented
ALB-origin integration and DNS cutover previously validated
DEV runtime currently OFF; no current live acceptance implied
```

---

## Summary

This infrastructure demonstrates a realistic DevOps/platform engineering workflow using AWS, Terraform, EKS, Kubernetes, Helm, ArgoCD, GitLab CI/CD, AWS Secrets Manager, RDS PostgreSQL, ingress, TLS, and documented architecture decisions.
