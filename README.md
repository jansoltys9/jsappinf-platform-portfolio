# JSAPPINF – AWS EKS DevOps Platform Lab

JSAPPINF is a production-like DevOps platform lab built on AWS EKS.
The project demonstrates practical platform engineering work across infrastructure as code, Kubernetes, GitOps, secure secrets delivery, application deployment, ingress, TLS, database integration, and operational troubleshooting.

The goal of this repository is to present a sanitized portfolio view of the project without exposing private credentials, Terraform state, environment-specific secrets, or sensitive account configuration.

---

## Visual Platform Overview

A visual guide to the platform infrastructure, repository landscape, deployment process, responsibility boundaries and application architecture is available here:

[Open the JSAPP Platform Visual Overview](docs/visual-platform-overview.md)

## Project Summary

The platform is built around AWS EKS and provisioned with Terraform.

As of 2026-09-17, DEV runtime is intentionally OFF for cost control. Runtime descriptions below refer to source configuration or previously validated capabilities, not currently running workloads. The retained foundation includes VPC/subnets/routing, the S3 gateway endpoint, Terraform backend/state, selected KMS keys, ECR, Route 53, Cognito and repository credentials; it does not include the removed EKS, RDS, RabbitMQ broker, NAT gateway, interface endpoints or ALBs. This status is based on recorded source/operator evidence, not a new live inventory.

## Repository Model

The real project is intentionally split across multiple repositories. This mirrors a production-like ownership model where infrastructure provisioning, application source code, Helm packaging and GitOps runtime configuration are separated.

| Repository | Purpose | Public Portfolio Status |
|---|---|---|
| `terraform-modules` | Reusable Terraform modules such as VPC, EKS, RDS, ECR and supporting infrastructure modules. | Private real repo, described only. |
| `jsappinf-platform` | Environment-level infrastructure composition, platform add-ons, component registry, IAM/IRSA, KMS, DNS, Gateway API, identity, database provisioning and edge-security wiring. | Private real repo, described only. |
| `JSAPP` | Node.js microservices source code for UI, user, product and order services. GitLab CI builds immutable images and pushes them to AWS ECR. The application owns database contracts, publishers and consumers. | Private real repo, described only. |
| `helmchartsappjs` | Reusable Helm charts for JSAPP services, including deployments, services, legacy optional Ingress templates, scheduling, probes and runtime configuration. Current Gateway API routes live in GitOps. | Private real repo, described only. |
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

docs/platform-architecture.md#7-secrets-architecture
  AWS Secrets Manager, External Secrets Operator and Kubernetes runtime secret delivery.

docs/multi-env-and-account-strategy.md
  Planned dev/stage/prod-like environment model and lightweight AWS account strategy.

docs/edge-security-cloudfront-waf.md
  CloudFront + AWS WAF edge security direction and clean public routing model.

docs/karpenter-capacity-flow.md
  Optional Karpenter ON_DEMAND/SPOT capacity validation model.

docs/foundation-and-lightweight-landing-zone-strategy.md
  Foundation-layer and lightweight Landing Zone strategy for multi-environment growth.

docs/component-responsibility-matrix.md
  Component ownership, repository boundaries, Terraform state ownership and lifecycle classification.

docs/platform-architecture.md
  AWS platform architecture, routing, identity, messaging, database and lifecycle design with validation boundaries.

docs/cross-cloud-platform-equivalence-strategy.md
  AWS, Azure and Google Cloud equivalence strategy and portable platform boundaries.
```

---


It includes:

```text
AWS VPC
AWS EKS
Managed node groups
AWS ECR
AWS RDS PostgreSQL
RDS application-provisioner Lambda
RabbitMQ application messaging
Amazon Cognito
AWS Secrets Manager
AWS KMS responsibility model
Route 53 DNS
cert-manager
External Secrets Operator
AWS Load Balancer Controller
Kubernetes Gateway API
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
       |
       +--> RDS PostgreSQL
       +--> RabbitMQ messaging
       +--> Secrets Manager through External Secrets
       +--> Cognito identity and JWT contract
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
  manages TLS certificates where required

AWS Load Balancer Controller:
  provisions and manages the Application Load Balancer integration

Kubernetes Gateway API:
  defines Gateway and HTTPRoute application routing

RDS PostgreSQL:
  provides application database persistence

RDS application-provisioner Lambda:
  creates or reconciles service users, schemas, grants and generated credentials

RabbitMQ:
  delivers the validated order.created event from order-service to product-service

Amazon Cognito:
  provides the validated identity infrastructure, PKCE flow and JWT contract;
  UI and backend integration exists in source; current-source live acceptance remains pending

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

The runtime database-secret model is:

```text
Terraform
  -> RDS application-provisioner Lambda
  -> service database users, schemas and grants
  -> generated credentials in AWS Secrets Manager
  -> External Secrets Operator
  -> Kubernetes Secret
  -> application pod environment variables
  -> RDS PostgreSQL login
```

The provisioner owns database identities and permissions. Application migrations and seed jobs own tables, indexes and initial application data.

D01 source implementation, canonical integration and local validation are complete; live acceptance has not yet been performed. Verified PostgreSQL TLS uses an explicitly mounted RDS CA bundle and fails closed on missing or invalid trust.

C01 immutable migration/seed generation and full-sync lifecycle gates are implemented in source. DEV remains OFF; current live execution is not claimed.

Example database runtime contract (credentials from Secrets; TLS settings from workload configuration):

```text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
DB_SSL=true
DB_SSL_REJECT_UNAUTHORIZED=true
DB_SSL_CA_FILE=/etc/ssl/db-ca/ca.pem
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

Validated synchronous application capabilities:

```text
Browser / UI
  -> ui-service
       |
       +--> user-service
       |      -> users schema
       |
       +--> product-service
       |      -> products schema
       |
       +--> order-service
              -> orders schema
```

Validated asynchronous messaging flow:

```text
order-service
  -> publishes order.created
  -> RabbitMQ
  -> product-service consumer
```

The application validates Kubernetes deployment, internal service discovery, database connectivity, runtime secret delivery, Gateway API routing and the core publisher-to-consumer messaging path.

---

## Edge Security Direction

A CloudFront + AWS WAF edge security layer has been designed and its infrastructure and cutover workflow have been validated.

Current target direction:

```text
Internet
  -> CloudFront
  -> AWS WAF Web ACL
  -> Application Load Balancer
  -> AWS Load Balancer Controller
  -> Kubernetes Gateway API
  -> HTTPRoute
  -> JSAPP services
```

The previous NLB, ingress-nginx and Kubernetes Ingress path has been replaced by the ALB and Gateway API model.

Gateway and HTTPRoute resources describe routing configuration; the AWS Load Balancer Controller reconciles that configuration into the ALB. They are not additional network hops in the request path.

Current implementation status:

```text
CloudFront and AWS WAF infrastructure implemented
ALB origin integration validated
edge cutover workflow validated
Gateway API application routing validated
long-running edge exposure disabled when not required
```

The edge layer is intentionally cost-aware in the development environment. It can be enabled for validation and presentation without being kept active continuously.

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
RabbitMQ order.created publisher-to-consumer messaging
RDS application identity and secret provisioning
Secrets Manager and External Secrets Operator
Cognito PKCE and JWT validation
Route 53 DNS and TLS automation
ALB and Kubernetes Gateway API routing
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
