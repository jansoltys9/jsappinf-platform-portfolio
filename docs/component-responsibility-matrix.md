# Component Responsibility Matrix

## Purpose

This document defines ownership boundaries for the main JSAPPINF platform components.

The matrix connects:

```text
component registry
Terraform module ownership
Terraform state ownership
environment wiring
Helm packaging
GitOps configuration
application implementation
runtime operations
lifecycle and cost behavior
```

The goal is to prevent unclear ownership, duplicated resources and Terraform state conflicts.

The current matrix describes the close-to-production development platform implementation. The same ownership principles are intended to scale toward separate environments, Terraform states and AWS workload accounts.

---

## Ownership Principles

Each component must have one clear infrastructure owner.

A resource must not be created independently by multiple Terraform root stacks.

```text
one resource
one state owner
one primary repository owner
```

Other repositories may consume outputs or configuration, but they must not recreate the same infrastructure resource.

The ownership model distinguishes:

```text
module ownership
state ownership
runtime ownership
application ownership
```

These responsibilities may belong to different repositories or teams.

---

## Lifecycle Classification

### Persistent

Resources that can normally remain deployed between development sessions because they have low or usage-based cost.

Examples:

```text
Cognito User Pool
IAM roles and policies
Route 53 records
ACM certificates
SSM standard parameters
small S3 state objects
```

### Pausable

Resources that may remain provisioned but can be operationally disabled or left idle.

Examples:

```text
Cognito test users
Lambda functions without invocations
EventBridge rules
SQS queues
SNS topics
```

### Ephemeral

Resources that normally should be destroyed or disabled when the development environment is not in use.

Examples:

```text
EKS cluster
managed node groups
NAT Gateway
RDS
Application Load Balancer
VPC interface endpoints
RabbitMQ runtime workloads
CloudFront and WAF development configuration when not required
```

Lifecycle classification does not replace Terraform ownership.

A persistent resource still requires one dedicated state owner.

---

## Responsibility Matrix

| Registry key / capability | Component | Terraform module owner | State / root-stack owner | Environment wiring | Helm owner | GitOps owner | Application owner | Runtime owner | Lifecycle | Current status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `vpc` | AWS VPC, subnets, routes, NAT and flow logs | `terraform-modules` | `jsappinf-platform/envs/dev/infra-next` | `jsappinf-platform` | — | — | — | Platform | Ephemeral | Implemented and validated |
| `eks` | Amazon EKS control plane and managed node groups | `terraform-modules` | `jsappinf-platform/envs/dev/infra-next` | `jsappinf-platform` | — | — | — | Platform | Ephemeral | Implemented and validated |
| `ecr` | Application container registries | `terraform-modules` | `jsappinf-platform/envs/dev/infra-next` | `jsappinf-platform` | — | — | `JSAPP` publishes images | Platform | Persistent or low-cost | Implemented and validated |
| `rds` | PostgreSQL database | `terraform-modules` | `jsappinf-platform/envs/dev/infra-next` | `jsappinf-platform` | `helmchartsappjs` connection configuration | `jsappinf-gitops` secrets and runtime values | `JSAPP` schemas, queries and migrations | Shared platform/application | Ephemeral | Implemented and validated |
| `rds-app-provisioner` | Lambda-based database user, schema, grant and secret provisioning | `jsappinf-platform` project module | `jsappinf-platform/envs/dev/infra-next` | `jsappinf-platform` | — | consumes generated Secrets Manager values through `jsappinf-gitops` | `JSAPP` defines required service database contracts | Shared platform/application | Pausable | Provisioning implemented and validated; automated credential rotation planned |
| `aws-load-balancer-controller` | AWS Load Balancer Controller, IAM and IRSA | `jsappinf-platform` | `jsappinf-platform` infrastructure state plus Kubernetes runtime state | `jsappinf-platform` | upstream Helm chart with platform values | `jsappinf-gitops` | — | Platform | Ephemeral with EKS | Implemented and validated |
| `gateway-api` | Gateway API resources and ALB routing | `jsappinf-platform` | Kubernetes runtime state reconciled by `jsappinf-gitops` | `jsappinf-platform` prerequisites | `helmchartsappjs` for reusable route shape | `jsappinf-gitops` | `JSAPP` services exposed through `helmchartsappjs` | Platform | Ephemeral with EKS | Implemented and validated |
| `external-dns` | Route 53 DNS automation | `jsappinf-platform` | `jsappinf-platform` IAM state plus Kubernetes runtime state | `jsappinf-platform` | upstream Helm chart with platform values | `jsappinf-gitops` | — | Platform | Ephemeral with EKS | Implemented and validated |
| `cert-manager` | Kubernetes certificate automation | `jsappinf-platform` | Kubernetes runtime state reconciled by `jsappinf-gitops` | `jsappinf-platform` prerequisites | upstream Helm chart with platform values | `jsappinf-gitops` | — | Platform | Ephemeral with EKS | Optional / currently disabled where not required |
| `external-secrets` | External Secrets Operator and AWS integration | `jsappinf-platform` | `jsappinf-platform` IAM state plus Kubernetes runtime state | `jsappinf-platform` | upstream Helm chart with platform values | `jsappinf-gitops` owns ExternalSecret and SecretStore resources | `JSAPP` consumes injected values | Platform | Ephemeral with EKS | Implemented and validated |
| `argocd` | GitOps reconciliation platform | `jsappinf-platform` | Kubernetes runtime state bootstrapped by `jsappinf-platform` | `jsappinf-platform` bootstrap | upstream ArgoCD Helm chart with platform values | `jsappinf-gitops` owns desired-state reconciliation | — | Platform | Ephemeral with EKS | Implemented and validated |
| `karpenter` | Dynamic Kubernetes capacity | `jsappinf-platform` | `jsappinf-platform` IAM state plus Kubernetes runtime state | `jsappinf-platform` | upstream Helm chart plus platform NodePool templates | `jsappinf-gitops` | `helmchartsappjs` provides workload labels and scheduling requirements | Platform | Ephemeral | Implemented and validated as optional capacity model |
| `rabbitmq` | Messaging platform capability | `jsappinf-platform` | `jsappinf-platform` infrastructure state or Kubernetes runtime state | `jsappinf-platform` | platform RabbitMQ packaging | `jsappinf-gitops` owns runtime application and environment values | `JSAPP` owns event schemas, publishers and consumers | Shared platform/application | Ephemeral | Infrastructure and core application messaging flow implemented and validated |
| `edge-waf` | CloudFront and AWS WAF edge layer | `jsappinf-platform` project module | `jsappinf-platform/envs/dev/infra-next` | `jsappinf-platform` | — | — | — | Platform/security | Ephemeral in dev | Infrastructure and cutover workflow validated |
| `amazon-cognito` | Cognito User Pool, app client, groups, resource server and domain | `jsappinf-platform/modules/identity/amazon-cognito` | `jsappinf-platform/envs/dev/identity-cognito` | dedicated identity root stack | `helmchartsappjs` public runtime configuration shape | `jsappinf-gitops` environment-specific public values | `JSAPP` UI login and backend JWT middleware | Shared identity/platform | Persistent | Infrastructure, PKCE and JWT validation complete; application integration planned |
| `secrets-manager` | Runtime secret source | `terraform-modules` and `jsappinf-platform` project modules | owning `jsappinf-platform` infrastructure state | `jsappinf-platform` | `helmchartsappjs` secret-reference shape | `jsappinf-gitops` owns ExternalSecret definitions | `JSAPP` consumes secrets | Platform | Persistent with usage-based cost | Implemented and validated |
| `kms` | Encryption responsibility and state/data keys | `terraform-modules` and `jsappinf-platform` project modules | owning `jsappinf-platform` infrastructure stack | `jsappinf-platform` | — | `jsappinf-gitops` contains references only | — | Platform/security | Persistent | Implemented according to component ownership |
| `route53` | Public DNS records and zones | `terraform-modules` and `jsappinf-platform` project modules | owning `jsappinf-platform` or future foundation stack | `jsappinf-platform` | — | `jsappinf-gitops` through ExternalDNS for selected records | — | Platform | Persistent | Implemented and validated |
| `ui-service` | Browser-facing Node.js service | — | ECR and infrastructure dependencies only | ECR, DNS and routing dependencies | `helmchartsappjs` | `jsappinf-gitops` | `JSAPP` | Application | Ephemeral runtime | Application runtime validated; Cognito integration planned |
| `user-service` | User/profile application service | — | ECR and infrastructure dependencies only | ECR, database and secret dependencies | `helmchartsappjs` | `jsappinf-gitops` | `JSAPP` | Application | Ephemeral runtime | Application and database flow validated |
| `product-service` | Product application service | — | ECR and infrastructure dependencies only | ECR, database, messaging and secret dependencies | `helmchartsappjs` | `jsappinf-gitops` | `JSAPP` | Application | Ephemeral runtime | Application, database flow and RabbitMQ consumer validated |
| `order-service` | Order application service | — | ECR and infrastructure dependencies only | ECR, database, messaging and secret dependencies | `helmchartsappjs` | `jsappinf-gitops` | `JSAPP` | Application | Ephemeral runtime | Application, database flow and RabbitMQ publisher validated |
| `monitoring` | Prometheus and Grafana | `jsappinf-platform` runtime configuration | Kubernetes runtime state reconciled by `jsappinf-gitops` | `jsappinf-platform` prerequisites | upstream monitoring charts with platform values | `jsappinf-gitops` | `JSAPP` exposes application metrics and health endpoints | Platform | Ephemeral with EKS | Implemented as runtime capability |

---

## Repository Responsibilities

### `terraform-modules`

Owns broadly reusable infrastructure modules.

Examples:

```text
VPC
EKS
RDS
ECR
bastion and supporting infrastructure modules
```

It must not contain environment-specific account identifiers, private DNS values or runtime application configuration.

### `jsappinf-platform`

Owns:

```text
environment composition
Terraform root stacks
component registry
project-shared platform modules
IAM and IRSA
networking integration
identity infrastructure
edge infrastructure
platform standards
Terraform state boundaries
```

It does not own application business logic.

### `JSAPP`

Owns:

```text
Node.js application source code
service APIs
database queries
Dockerfiles
application tests
CI image-build logic
RabbitMQ producers and consumers
Cognito UI and backend integration
```

It does not provision AWS infrastructure.

### `helmchartsappjs`

Owns reusable Kubernetes workload packaging:

```text
Deployments
Services
probes
autoscaling
scheduling configuration
environment-variable templates
Gateway API route templates where reusable
authentication configuration shape
```

It must not hard-code development account identifiers or secret values.

### `jsappinf-gitops`

Owns environment-specific desired runtime state:

```text
ArgoCD Applications
environment-specific Helm values
namespaces
platform applications
ExternalSecrets
Gateway and HTTPRoute configuration
runtime feature flags
image versions
```

It must not provision foundational AWS infrastructure.

### `jsappinf-platform-portfolio`

Owns only sanitized public documentation.

It must not contain:

```text
AWS account IDs
Terraform state
backend configuration
private tfvars
credentials
tokens
passwords
private repository URLs
sensitive infrastructure identifiers
```

---

## Terraform State Ownership

Current principal state boundaries:

```text
main development platform:
envs/dev/infra-next

persistent Cognito identity:
envs/dev/identity-cognito
```

The Cognito root stack exclusively owns:

```text
User Pool
app client
managed domain
resource server
groups
```

The main platform stack must not recreate these resources.

Application, Helm and GitOps repositories consume Cognito public configuration but do not own the Cognito resources.

---

## Gateway API Ownership

The current routing direction uses:

```text
AWS Load Balancer Controller
Gateway API
Gateway
HTTPRoute
Application Load Balancer
```

Responsibility split:

```text
AWS and IAM prerequisites:
jsappinf-platform

controller installation and configuration:
platform Helm/GitOps configuration

reusable route template shape:
helmchartsappjs where appropriate

environment hostnames and path rules:
jsappinf-gitops

application Services and ports:
JSAPP plus helmchartsappjs
```

The older ingress-nginx and NLB model is considered a previous platform iteration, not the current target architecture.

---

## Cognito Ownership

Infrastructure ownership:

```text
jsappinf-platform
```

Application responsibilities:

```text
UI Authorization Code + PKCE flow
callback handling
token lifecycle
backend JWT verification
scope authorization
group authorization
```

Runtime configuration responsibilities:

```text
Helm:
reusable authentication values and environment variables

GitOps:
development-specific issuer, client ID, domain and scopes
```

The public browser client does not use a client secret.

Identity ownership boundary:

```text
Cognito:
authentication identity
credential and MFA policies
federation
token issuance
groups and OAuth scopes

JSAPP backend services:
JWT validation
scope and group authorization
business authorization

user-service:
application profile
business attributes
preferences
application-specific user state
```

The application profile should reference the Cognito `sub` claim as the stable external identity key.

A separate local LDAP or custom token-issuing authentication service is not currently required.

Corporate identity integration, if needed later, should preferably use OIDC or SAML federation through Cognito rather than direct LDAP integration in application services.

---

## RabbitMQ Ownership

Platform responsibilities:

```text
RabbitMQ deployment
networking
credentials delivery
availability configuration
monitoring
queue-policy foundations
```

Application responsibilities:

```text
event schema
publisher implementation
consumer implementation
acknowledgement behavior
retry logic
dead-letter handling
idempotency
business processing
```

The core application messaging flow is implemented and validated through the order-service publisher, RabbitMQ event delivery and product-service consumer.

Additional events, consumers, retry policies, dead-letter handling and recovery testing remain future enhancements.

---

## Edge Security Ownership

Platform and security responsibilities:

```text
CloudFront distribution
AWS WAF Web ACL
managed rules
rate-based rules
origin configuration
DNS cutover
rollback procedure
metrics and logging decisions
```

Application responsibilities:

```text
correct cache-control behavior
public and private route behavior
safe API semantics
health endpoints
```

The target edge request path is:

```text
Internet
  -> CloudFront
  -> AWS WAF
  -> Application Load Balancer
  -> Gateway API
  -> HTTPRoute
  -> Kubernetes Service
  -> application Pod
```

---

## Status Terminology

Portfolio documents should use the following terminology consistently.

### Implemented and validated

The component was provisioned and tested in a running environment.

### Infrastructure validated

The infrastructure component was created and technically verified, but application-level integration may still be incomplete.

### Prepared

Terraform, Helm, GitOps or design foundations exist, but full runtime validation is not complete.

### Planned

The work is documented but not yet implemented.

### Previous iteration

The approach was used earlier but has been superseded by the current target architecture.

---

## Review Checklist

Before documenting a component as complete, verify:

```text
Does the resource exist in the correct Terraform state?
Was the infrastructure applied successfully?
Was runtime behavior tested?
Was application integration tested?
Is the component currently active or only validated previously?
Is the lifecycle class documented?
Is the owner repository clear?
Are secrets and identifiers sanitized?
```
