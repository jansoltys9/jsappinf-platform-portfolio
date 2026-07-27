# Multi-Environment and AWS Account Strategy

This document describes the planned evolution of the JSAPPINF platform from a single dev environment into a multi-environment and lightweight account-aware AWS architecture.

The goal is to demonstrate production-like thinking without overengineering the lab.

---

## Current State

The current platform focuses on one main development environment.

```text
envs/dev/infra-next
```

The dev environment supports:

```text
Terraform infrastructure
AWS EKS
ArgoCD
Helm-based microservices
External Secrets Operator
AWS Secrets Manager
RDS PostgreSQL
cert-manager
ingress-nginx
Route 53 DNS
ECR
monitoring
optional Karpenter
CloudFront + AWS WAF design
```

This environment is designed to be rebuilt and destroyed frequently to control cost.

---

## Target Environment Model

The planned environment model is:

```text
dev
stage
prod-sim
```

The goal is not to create real production immediately, but to show a realistic path toward environment separation.

---

## dev Environment

Purpose:

```text
daily development
experimentation
cost-aware rebuild/destroy workflow
fast validation
lower baseline capacity
```

Typical behavior:

```text
short-lived environment
minimal cost profile
Karpenter disabled or idle by default
SPOT burst disabled by default
lower node count preferred
```

The dev environment is where most experiments and platform iterations happen.

---

## stage Environment

Purpose:

```text
integration validation
more stable testing profile
closer-to-real GitOps validation
application and platform integration checks
```

Possible stage behavior:

```text
longer-lived than dev
more stable Terraform state
full ArgoCD application deployment
External Secrets validation
Ingress/TLS validation
optional Karpenter validation
```

The stage environment should be less experimental than dev.

---

## prod-sim Environment

Purpose:

```text
production-like simulation
cleaner architecture profile
protected resources
stricter defaults
fewer experiments
```

This is not real production, but it demonstrates production-style separation and platform discipline.

Possible prod-sim behavior:

```text
deletion protection enabled for selected resources
stronger security defaults
more conservative capacity model
controlled changes
separate state
separate secrets
stronger access boundaries
```

---

## Backend State Separation

Each environment should have its own Terraform backend state key.

Example:

```text
dev:
  key = envs/dev/infra-next/terraform.tfstate

stage:
  key = envs/stage/infra-next/terraform.tfstate

prod-sim:
  key = envs/prod-sim/infra-next/terraform.tfstate
```

This avoids accidental state overlap and supports safer environment lifecycle management.

---

## Environment Configuration

Each environment should define its own values for:

```text
environment name
name prefix
AWS region
backend state key
tags
enabled components
cost profile
capacity profile
deletion protection behavior
public access behavior
domain/subdomain strategy
secret paths
```

Example naming:

```text
jsappinf-dev
jsappinf-stage
jsappinf-prod-sim
```

---

## Suggested Folder Direction

A simple and readable structure could be:

```text
envs/
  dev/
    infra-next/
      backend.hcl
      terraform.tfvars
      main.tf
      variables.tf

  stage/
    infra-next/
      backend.hcl
      terraform.tfvars
      main.tf
      variables.tf

  prod-sim/
    infra-next/
      backend.hcl
      terraform.tfvars
      main.tf
      variables.tf
```

The first implementation can reuse the same Terraform composition with different environment-specific inputs.

---

## Component Enablement by Environment

Not every component must be enabled in every environment.

Example:

```text
dev:
  Karpenter optional or disabled
  Edge WAF disabled by default
  lower node count
  short-lived RDS
  deletion protection disabled

stage:
  ArgoCD enabled
  apps enabled
  secrets enabled
  ingress/TLS enabled
  optional Karpenter validation
  Edge WAF optional

prod-sim:
  stronger defaults
  selected deletion protection
  Edge WAF enabled
  stricter secret and DNS behavior
  fewer experimental flags
```

---

## AWS Account Direction

The project can start with multiple environments in one AWS account.

Later, it can evolve toward a lightweight multi-account model.

Possible future account layout:

```text
Management account:
  billing
  account management
  future IAM Identity Center

NonProd account:
  dev
  stage
  experiments

ProdSim account:
  production-like simulation
  cleaner profile
  protected resources
```

The Management account should not host workloads.

---

## Lightweight Landing Zone Direction

The project does not need a full enterprise Landing Zone at the current stage.

Not immediate:

```text
full AWS Control Tower
central security account
central logging account
Transit Gateway
cross-account shared services VPC
complex SCP structure
multi-region production topology
```

Reasonable lightweight Landing Zone concepts to introduce later:

```text
account separation
naming and tagging standards
centralized billing awareness
backend state separation
baseline IAM strategy
baseline logging strategy
baseline security guardrails
environment-specific access model
```

The goal is to show that the platform is designed with production direction in mind, while keeping the implementation realistic for a portfolio lab.

---

## Secrets by Environment

Secrets should be environment-specific.

Example paths:

```text
jsappinf/dev/app/db/ui-service
jsappinf/stage/app/db/ui-service
jsappinf/prod-sim/app/db/ui-service
```

This prevents accidental reuse of dev credentials in stage or prod-sim.

---

## DNS and Public Routing by Environment

The current lab model may expose individual services through separate hostnames because it is useful for learning, testing and troubleshooting.

Current lab/debug model:

```text
ui.jsapp365.com
user.jsapp365.com
product.jsapp365.com
order.jsapp365.com
```

This model is practical during early development because every service can be tested directly.

However, the target production-like model should not expose every small backend microservice as a separate public DNS name.

The cleaner target model is:

```text
one public product entrypoint
path-based API routing
microservices hidden behind the ingress layer
```

Recommended target DNS and routing model:

```text
dev:
  dev.jsapp365.com
  dev.jsapp365.com/api/users
  dev.jsapp365.com/api/products
  dev.jsapp365.com/api/orders

stage:
  stage.jsapp365.com
  stage.jsapp365.com/api/users
  stage.jsapp365.com/api/products
  stage.jsapp365.com/api/orders

prod / prod-like public:
  jsapp365.com
  jsapp365.com/api/users
  jsapp365.com/api/products
  jsapp365.com/api/orders
```

Recommended production-like routing:

```text
jsapp365.com
  -> frontend / UI

jsapp365.com/api/users
  -> user-service

jsapp365.com/api/products
  -> product-service

jsapp365.com/api/orders
  -> order-service
```

Recommended dev routing:

```text
dev.jsapp365.com
  -> frontend / UI

dev.jsapp365.com/api/users
  -> user-service

dev.jsapp365.com/api/products
  -> product-service

dev.jsapp365.com/api/orders
  -> order-service
```

The production-like public domain should not expose labels such as `prod`, `prod-sim`, or similar environment markers in the user-facing hostname.

Environment identity should be handled internally through:

```text
Terraform backend keys
AWS accounts
tags
name prefixes
secret paths
ArgoCD environment configuration
deployment namespaces
```

Microservices should remain an internal implementation detail.

Internal Kubernetes service discovery can still use service-level DNS names such as:

```text
user-service.user-service.svc.cluster.local
product-service.product-service.svc.cluster.local
order-service.order-service.svc.cluster.local
ui-service.ui-service.svc.cluster.local
```

Recommended evolution:

```text
Phase 1:
  service-per-subdomain for lab testing and debugging

Phase 2:
  single dev entrypoint with /api path routing

Phase 3:
  single stage entrypoint with /api path routing

Phase 4:
  clean production-like public entrypoint

Phase 5:
  CloudFront + AWS WAF in front of clean public entrypoints
```

This keeps the current lab practical while showing a clear direction toward a more realistic public routing model.

---

## Cost Model

The project should stay cost-aware.

Cost-aware choices:

```text
destroy dev when not used
keep Karpenter idle by default
disable burst capacity by default
use small instance classes in dev
avoid unnecessary multi-AZ services in dev
avoid expensive optional WAF features initially
```

Production-like behavior can be demonstrated without keeping all expensive components running continuously.

---

## Implementation Roadmap

Recommended order:

```text
1. Document environment model.
2. Define dev/stage/prod-sim profiles.
3. Separate backend keys.
4. Create stage tfvars.
5. Create prod-sim tfvars.
6. Standardize naming and tags.
7. Standardize secret paths.
8. Decide account separation timing.
9. Add lightweight Landing Zone baseline.
```

---

## Portfolio Summary

The multi-environment strategy shows a realistic evolution path:

```text
single dev EKS lab
  -> dev/stage/prod-sim environments
  -> separate state and configuration
  -> environment-specific secrets and DNS
  -> future AWS account separation
  -> lightweight landing-zone-style standards
```

This demonstrates that the platform is not only a working dev environment, but also has a clear direction toward production-like structure.
