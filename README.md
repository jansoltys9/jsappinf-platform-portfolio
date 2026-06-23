# JSAPPINF – AWS EKS DevOps Platform Lab

JSAPPINF is a production-like DevOps platform lab built on AWS EKS.  
The project demonstrates practical platform engineering work across infrastructure as code, Kubernetes, GitOps, secure secrets delivery, application deployment, ingress, TLS, database integration, and operational troubleshooting.

The goal of this repository is to present a sanitized portfolio view of the project without exposing private credentials, Terraform state, environment-specific secrets, or sensitive account configuration.

---

## Project Summary

The platform is built around AWS EKS and provisioned with Terraform.

## Repository Model

The real project is intentionally split across multiple repositories. This mirrors a production-like ownership model where infrastructure provisioning, application source code, Helm packaging and GitOps runtime configuration are separated.

| Repository | Purpose | Public Portfolio Status |
|---|---|---|
| `terraform-modules` | Reusable Terraform modules such as VPC, EKS, RDS, ECR and supporting infrastructure modules. | Private real repo, described only. |
| `jsappinf-platform` | Environment-level infrastructure composition, platform add-ons, component registry, IAM/IRSA, KMS, DNS, ingress and edge-security wiring. | Private real repo, described only. |
| `JSAPP` | Node.js microservices source code for UI, user, product and order services. GitLab CI builds immutable images and pushes them to AWS ECR. | Private real repo, described only. |
| `helmchartsappjs` | Helm charts for JSAPP services, including deployment, service, ingress and runtime configuration templates. | Private real repo, described only. |
| `jsappinf-gitops` | ArgoCD desired runtime state, app-of-apps model, platform applications, ExternalSecrets and environment-specific deployment configuration. | Private real repo, described only. |
| `jsappinf-platform-portfolio` | Sanitized public documentation repository for portfolio and LinkedIn presentation. It does not contain secrets, Terraform state, private credentials or sensitive account configuration. | Public portfolio repo. |

The private repositories contain the real implementation. This public portfolio repository documents the architecture, workflows and decisions without exposing sensitive project data.

---

## GitLab vs GitHub Publication Model

The real implementation repositories are kept private in GitLab.

GitLab is used as the primary working platform for:

```text
real infrastructure code
real application source code
real Helm charts
real GitOps manifests
real CI/CD pipelines
private project history
```

The public GitHub repository is intentionally sanitized and documentation-focused.

GitHub is used as the public portfolio layer for:

```text
architecture overview
repository ownership model
GitOps deployment flow
secrets delivery model
multi-environment strategy
edge security design
capacity planning notes
LinkedIn Featured project link
```

This separation is intentional.

```text
GitLab:
  private implementation and day-to-day engineering work

GitHub:
  public portfolio documentation without sensitive data
```

The public repository does not include:

```text
Terraform state
backend configuration
tfvars files
private credentials
tokens
real secret values
AWS account-specific identifiers
private pipeline variables
sensitive GitOps repository secrets
```

This allows the project to be presented publicly while keeping the real implementation and operational details protected.

---


## Documentation

Detailed portfolio documentation is available in the `docs/` directory.

```text
docs/infrastructure-overview.md
  High-level AWS, EKS, networking, platform add-ons and application architecture.

docs/gitops-deployment-flow.md
  GitLab CI, ECR, Helm, ArgoCD and Kubernetes rollout flow.

docs/repository-ownership-model.md
  Repository responsibility boundaries across Terraform, application code, Helm and GitOps.

docs/secrets-flow.md
  AWS Secrets Manager, External Secrets Operator and Kubernetes runtime secret delivery.

docs/multi-env-and-account-strategy.md
  Planned dev/stage/prod-like environment model and lightweight AWS account strategy.

docs/edge-security-cloudfront-waf.md
  CloudFront + AWS WAF edge security direction and clean public routing model.

docs/karpenter-capacity-flow.md
  Optional Karpenter ON_DEMAND/SPOT capacity validation model.
```

---


It includes:

```text
AWS VPC
AWS EKS
Managed node groups
AWS ECR
AWS RDS PostgreSQL
AWS Secrets Manager
AWS KMS responsibility model
Route 53 DNS
cert-manager
External Secrets Operator
ingress-nginx
ArgoCD
Helm-based application deployment
GitLab CI/CD image build flow
Node.js microservices
optional Karpenter capacity model
CloudFront + AWS WAF edge security design
```

The application layer consists of multiple Node.js microservices:

```text
ui-service
user-service
product-service
order-service
```

The full application flow was validated from the UI through internal services into PostgreSQL-backed persistence.

---

## High-Level Architecture

```text
Developer / Operator
  -> GitLab repositories
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

Core platform components:

```text
Terraform:
  provisions infrastructure and platform foundations

VPC:
  provides public and private networking

EKS:
  runs platform components and application workloads

ArgoCD:
  reconciles GitOps desired state into Kubernetes

External Secrets Operator:
  syncs runtime secrets from AWS Secrets Manager into Kubernetes

cert-manager:
  manages TLS certificates

ingress-nginx:
  exposes HTTP/HTTPS application routes

RDS PostgreSQL:
  provides application database persistence

ECR:
  stores immutable service container images
```

---

## GitOps Deployment Flow

The intended deployment model separates infrastructure provisioning from application delivery.

```text
Application source change
  -> GitLab CI pipeline
  -> Docker image build
  -> Push image to AWS ECR
  -> Helm chart / values update
  -> GitOps desired state update
  -> ArgoCD sync
  -> Kubernetes rollout
```

Applications are not manually deployed with `kubectl` as the main workflow.  
The preferred workflow is Git-based and reconciled by ArgoCD.

---

## Repository Ownership Model

The project is split across multiple repositories to reflect production-like ownership boundaries.

```text
terraform-modules:
  reusable Terraform modules such as VPC, EKS, RDS and future modules

jsappinf-platform:
  environment wiring, platform add-ons, Terraform composition and standards

JSAPP:
  Node.js application source code, Dockerfiles and CI build logic

helmchartsappjs:
  Helm charts, templates, service values and workload deployment shape

jsappinf-gitops:
  ArgoCD Applications, app-of-apps structure and runtime desired state
```

This split separates:

```text
infrastructure lifecycle
application build lifecycle
Kubernetes packaging lifecycle
runtime deployment lifecycle
secret delivery model
platform standards
```

---

## Secrets Management

Raw secret values are not stored in GitOps manifests.

The runtime secrets model is:

```text
Terraform / bootstrap process
  -> AWS Secrets Manager
  -> External Secrets Operator
  -> Kubernetes Secret
  -> Application pod environment variables
  -> RDS PostgreSQL login
```

Example application secret contract:

```text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
```

Security principle:

```text
Base64 is encoding, not encryption.
Do not store raw secret values in Git repositories.
Do not document decoded Kubernetes Secret values.
```

---

## Application Runtime

Current service ports:

```text
ui-service       3003
user-service     3000
product-service  3001
order-service    3002
```

Validated service flow:

```text
Browser / UI
  -> ui-service
  -> order-service
  -> user-service
  -> product-service
  -> RDS PostgreSQL
```

The application validates not only Kubernetes deployment, but also internal service discovery, database connectivity, runtime secrets, ingress routing, and end-to-end business flow.

---

## Edge Security Direction

A CloudFront + AWS WAF edge security layer has been designed for the platform.

Target direction:

```text
Internet
  -> CloudFront
  -> AWS WAF Web ACL
  -> existing NLB
  -> ingress-nginx
  -> Kubernetes Ingress
  -> JSAPP services
```

Current implementation status:

```text
design documented
component registry entry created
Terraform skeleton prepared
root wiring added in disabled mode
enabled Terraform plan validated
apply/testing intentionally deferred
```

The initial WAF baseline is intentionally cost-aware:

```text
AWSManagedRulesCommonRuleSet
AWSManagedRulesKnownBadInputsRuleSet
basic rate-based rule
no Bot Control initially
no CAPTCHA initially
no full WAF logging initially
```

---

## Capacity and Cost-Aware Dev Model

The project is designed as a cost-aware development environment.

Normal dev mode:

```text
lower baseline capacity
daily rebuild/destroy workflow
Karpenter disabled or idle by default
SPOT burst disabled by default
```

Validation mode:

```text
optional Karpenter testing
ON_DEMAND and SPOT application placement
stable/burst workload model
NodePool and scheduling validation
```

This keeps the project practical while still demonstrating production-like capacity design.

---

## Multi-Environment Direction

The current implementation focuses on a dev environment.

Planned evolution:

```text
dev:
  cost-aware daily rebuild profile

stage:
  integration validation profile

prod-sim:
  production-like simulation profile
```

Future direction:

```text
separate backend keys
separate tfvars
environment-specific naming and tags
future AWS Organization account separation
NonProd and ProdSim account boundaries
light landing-zone-style standards
```

The goal is not to overengineer the lab, but to show a realistic path from a single dev platform toward multi-environment and account-aware architecture.

---

## What This Project Demonstrates

This project demonstrates practical experience with:

```text
AWS platform engineering
Terraform infrastructure provisioning
EKS platform setup
Kubernetes workload deployment
Helm chart management
ArgoCD GitOps
GitLab CI/CD
ECR image delivery
RDS PostgreSQL integration
Secrets Manager and External Secrets Operator
Route 53 DNS and TLS automation
ingress-nginx routing
cost-aware environment lifecycle
production-like architecture documentation
operational troubleshooting
```

---

## Portfolio Note

This repository is a sanitized portfolio representation of the JSAPPINF project.

It intentionally excludes:

```text
Terraform state
tfvars files
backend configuration
secret values
tokens
private credentials
AWS account-specific sensitive configuration
raw Kubernetes Secret values
```

The purpose is to show architecture, reasoning, standards and implementation direction without exposing sensitive infrastructure details.
