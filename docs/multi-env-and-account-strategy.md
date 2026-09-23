> Read [Phase 1 status](phase1-implementation-status.md) first. CURRENT / VALIDATED refers only to dated historical DEV evidence. The reusable multi-environment code is an IMPLEMENTED CANDIDATE / NOT DEPLOYED. STAGE/PROD, Landing Zone and Azure/GCP live operation are not validated; any foundation/cross-cloud roadmap below is TARGET / PROPOSED and not a Phase 1 dependency.

# Multi-Environment and AWS Account Strategy

This document describes the Phase 1 multi-environment candidate and its future account-aware AWS evolution.

The goal is to demonstrate production-like thinking without overengineering the lab.

---

## Current State

The reviewed Phase 1 candidate has one reusable AWS platform composition for explicit DEV/STAGE/PROD inputs. It is not deployed. Historical DEV evidence remains the only runtime evidence.

As of 2026-09-17, DEV runtime is intentionally OFF for cost control. Runtime descriptions below refer to source configuration or previously validated capabilities, not currently running workloads.

The current candidate composition root is `stacks/aws/platform`. The former `envs/dev/infra-next` path is retained only in source history and migration evidence; it is not the active composition root.

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
ALB and Gateway API
Route 53 DNS
ECR
monitoring
optional Karpenter
CloudFront + AWS WAF infrastructure and previously validated cutover
```

This environment is designed to be rebuilt and destroyed frequently to control cost.

---

## Implemented Candidate Environment Model

The undeployed Phase 1 candidate model is:

```text
dev
stage
prod
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

## prod Environment

Purpose:

```text
production-like simulation
cleaner architecture profile
protected resources
stricter defaults
fewer experiments
```

This is an undeployed production candidate profile, not proof of a production environment or production certification.

Candidate prod behavior:

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
  key = platform/dev/terraform.tfstate

stage:
  key = platform/stage/terraform.tfstate

prod:
  key = platform/prod/terraform.tfstate
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
jsappinf-prod
```

---

## Implemented Candidate Folder Direction

A simple and readable structure could be:

```text
stacks/aws/platform/             # one reusable composition
env/aws/dev.tfvars               # explicit environment inputs
env/aws/stage.tfvars
env/aws/prod.tfvars
backends/aws/*.platform.hcl.example
```

The implemented candidate reuses the same Terraform composition with different environment-specific inputs and isolated backend templates. Environment, AWS account, AWS region and backend identity remain independent selections.

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

prod:
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
jsappinf/prod/app/db/ui-service
```

This prevents accidental reuse of dev credentials in stage or prod.

---

## DNS and Public Routing by Environment

The earlier lab model exposed individual service hostnames for testing. Current source uses app.dev for the UI and api.dev with service paths; the single-host examples below remain future alternatives.

Previous lab/debug model:

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

Phase 1 disposition and later order:

```text
1. IMPLEMENTED CANDIDATE / NOT DEPLOYED: one composition, dev/stage/prod tfvars, isolated backend templates, naming/tags and environment-specific secret paths.
2. ACTIVATION GATE: recover and reconcile authoritative DEV backend, state and historic inputs; implement the fail-closed environment/account/region/backend selector; review an authorized real plan.
3. TARGET / PROPOSED: activate and accept DEV, then qualify STAGE and PROD independently with their own account, region, backend and runtime evidence.
4. TARGET / OPTIONAL: decide whether account governance justifies a lightweight Landing Zone. It is not a Phase 1 prerequisite.
```

---

## Portfolio Summary

The multi-environment strategy shows a realistic evolution path:

```text
single dev EKS lab
  -> one reusable dev/stage/prod candidate composition
  -> separate state and configuration
  -> environment-specific secrets and DNS
  -> future AWS account separation
  -> lightweight landing-zone-style standards
```

The previously validated DEV environment is recorded as OFF in dated source evidence. The multi-environment structure is an IMPLEMENTED CANDIDATE / NOT DEPLOYED; environment activation, account bindings and live acceptance remain TARGET / PROPOSED.
