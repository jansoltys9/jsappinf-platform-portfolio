> Read [Phase 1 status](phase1-implementation-status.md) first. CURRENT / VALIDATED refers only to dated historical DEV evidence. The reusable multi-environment code is an IMPLEMENTED CANDIDATE / NOT DEPLOYED. STAGE/PROD, Landing Zone and Azure/GCP live operation are not validated; any foundation/cross-cloud roadmap below is TARGET / PROPOSED and not a Phase 1 dependency.

# JSAPPINF Platform Architecture

## Purpose

This document presents the current high-level architecture of the JSAPPINF AWS EKS platform.

The platform evolved from an earlier ingress-nginx and Network Load Balancer implementation toward:

```text
AWS Load Balancer Controller
Application Load Balancer
Kubernetes Gateway API
Gateway
HTTPRoute
CloudFront
AWS WAF
Amazon Cognito
RabbitMQ
```

The earlier ingress-nginx architecture remains part of the project history but is no longer the target routing model.

---

## Architecture Status

As of 2026-09-17, DEV runtime is intentionally OFF for cost control. Runtime descriptions below refer to source configuration or previously validated capabilities, not currently running workloads.

### Implemented and validated

```text
AWS VPC
Amazon EKS
managed node groups
AWS Load Balancer Controller
Application Load Balancer
Gateway API routing
ArgoCD GitOps
Helm application deployment
External Secrets Operator
AWS Secrets Manager
RDS PostgreSQL
application schemas and seed data
Amazon ECR
Karpenter capacity model
Amazon Cognito infrastructure
Cognito Authorization Code + PKCE flow
Cognito JWT signature verification through JWKS
RabbitMQ application messaging flow
order-service RabbitMQ publisher
product-service RabbitMQ consumer
end-to-end order.created event delivery
```

### Infrastructure capability prepared or validated

```text
CloudFront and AWS WAF infrastructure and cutover workflow validated
Cognito groups and custom API scopes validated
```

### Source implementation and remaining acceptance

UI login/callback, backend JWT verification and scope/group authorization are implemented in source. Live acceptance of the current source remains pending. D01 source/integration/local validation and C01 immutable lifecycle controls are implemented; live acceptance/execution of those changes is not claimed.

Remaining validation and evolution:

```text
live acceptance of current authentication and session hardening
long-running CloudFront and WAF exposure
additional RabbitMQ consumers and business-event flows
retry and dead-letter queue policies
```

---

# 1. High-Level Architecture

```text
                                  ┌────────────────────────┐
                                  │ Developer / Operator   │
                                  └────────────┬───────────┘
                                               │
                                               v
                                  ┌────────────────────────┐
                                  │ Private GitLab repos   │
                                  │                        │
                                  │ Terraform              │
                                  │ Application source     │
                                  │ Helm charts            │
                                  │ GitOps desired state   │
                                  └───────┬────────┬───────┘
                                          │        │
                              Terraform   │        │ CI / GitOps
                                          │        │
                                          v        v
┌──────────────────────────────────── AWS Account ───────────────────────────────────┐
│                                                                                    │
│  ┌──────────────────┐       ┌───────────────────────────────────────────────────┐  │
│  │ Amazon Cognito   │       │ Public application path                           │  │
│  │                  │       │                                                   │  │
│  │ User Pool        │       │ Route 53                                          │  │
│  │ UI app client    │       │    │                                              │  │
│  │ Groups           │       │    v                                              │  │
│  │ OAuth scopes     │       │ CloudFront                                        │  │
│  │ Managed login    │       │    │                                              │  │
│  └────────┬─────────┘       │ AWS WAF                                           │  │
│           │                 │    │                                              │  │
│           │ OAuth / OIDC    │    v                                              │  │
│           │                 │ Application Load Balancer                         │  │
│           │                 │    │                                              │  │
│           │                 │ Gateway API                                       │  │
│           │                 │    │                                              │  │
│           │                 │ Gateway + HTTPRoute                               │  │
│           │                 └────┬──────────────────────────────────────────────┘  │
│           │                      │                                                 │
│           │                      v                                                 │
│  ┌────────┴──────────────────── Amazon EKS ─────────────────────────────────────┐  │
│  │                                                                             │  │
│  │ Platform components                                                         │  │
│  │                                                                             │  │
│  │ ┌─────────┐ ┌──────────────────┐ ┌─────────────────────┐ ┌───────────────┐ │  │
│  │ │ ArgoCD  │ │ External Secrets │ │ AWS Load Balancer   │ │ ExternalDNS   │ │  │
│  │ │         │ │ Operator         │ │ Controller          │ │               │ │  │
│  │ └─────────┘ └──────────────────┘ └─────────────────────┘ └───────────────┘ │  │
│  │                                                                             │  │
│  │ ┌────────────┐ ┌────────────────┐ ┌──────────────────┐                      │  │
│  │ │ Karpenter  │ │ cert-manager   │ │ Monitoring       │                      │  │
│  │ └────────────┘ └────────────────┘ └──────────────────┘                      │  │
│  │                                                                             │  │
│  │ Application workloads                                                       │  │
│  │                                                                             │  │
│  │ ┌────────────┐ ┌──────────────┐ ┌────────────────┐ ┌────────────────┐      │  │
│  │ │ UI service │ │ User service │ │ Product service│ │ Order service  │      │  │
│  │ └──────┬─────┘ └──────┬───────┘ └───────┬────────┘ └───────┬────────┘      │  │
│  │        │              │                 │                  │               │  │
│  │        └──────────────┴─────────────────┴──────────────────┘               │  │
│  │                                     │                                       │  │
│  │                              ┌──────┴──────┐                                │  │
│  │                              │ RabbitMQ    │                                │  │
│  │                              │ messaging   │                                │  │
│  │                              └─────────────┘                                │  │
│  └─────────────────────────────────────┬───────────────────────────────────────┘  │
│                                        │                                           │
│                                        v                                           │
│                              ┌───────────────────────┐                             │
│                              │ RDS PostgreSQL        │                             │
│                              │                       │                             │
│                              │ users schema          │                             │
│                              │ products schema       │                             │
│                              │ orders schema         │                             │
│                              └───────────────────────┘                             │
│                                                                                    │
│  Supporting services:                                                              │
│                                                                                    │
│  ECR · Secrets Manager · KMS · Route 53 · CloudWatch · S3 Terraform state          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

---

# 2. Infrastructure Provisioning Flow

Terraform owns AWS infrastructure and platform prerequisites.

```text
Terraform modules
      |
      v
Environment root stacks
      |
      +--> VPC
      +--> EKS
      +--> managed node groups
      +--> IAM and IRSA
      +--> RDS
      +--> ECR
      +--> Secrets Manager
      +--> Route 53
      +--> ALB prerequisites
      +--> CloudFront and WAF
      +--> Cognito
      +--> RDS application-provisioner Lambda
```

The RDS application-provisioner Lambda performs application-level database identity provisioning after the RDS infrastructure is available.

Validated Lambda responsibilities:

```text
connect to the application database
create or reconcile service schemas
create or reconcile service database users
apply schema grants
generate service credentials
store service connection secrets in AWS Secrets Manager
return per-service provisioning status
```

Validated services:

```text
user-service:
  database user: user_user
  schema: users

product-service:
  database user: product_user
  schema: products

order-service:
  database user: order_user
  schema: orders

ui-service:
  database user: ui_user
  schema: ui
```

The Lambda does not own application tables, indexes or seed data.

Principal Terraform state boundaries:

```text
stacks/aws/platform
  reusable DEV/STAGE/PROD candidate composition (not deployed from this root)

envs/dev/identity-cognito
  persistent Cognito identity infrastructure
```

Each AWS resource has one Terraform state owner.

The Cognito resources are not recreated by the main EKS stack.

---

# 3. Kubernetes Routing Architecture

## Previous iteration

The earlier platform iteration used:

```text
Internet
  -> Network Load Balancer
  -> ingress-nginx
  -> Kubernetes Ingress
  -> Kubernetes Service
```

This implementation was useful for the initial Kubernetes ingress, DNS and TLS implementation phase.

## Current architecture

The current routing model uses:

```text
Internet
  -> Application Load Balancer
  -> Gateway API
  -> Gateway
  -> HTTPRoute
  -> Kubernetes Service
  -> application Pod
```

The AWS Load Balancer Controller integrates Kubernetes Gateway API resources with an AWS Application Load Balancer.

Gateway and HTTPRoute resources describe routing configuration; the AWS Load Balancer Controller reconciles that configuration into the ALB. They are not additional network hops in the request path.

Responsibility split:

```text
Terraform:
  IAM, IRSA, networking and controller prerequisites

Helm:
  reusable Service packaging; current HTTPRoutes are GitOps manifests

GitOps:
  Gateway, HTTPRoute, hostnames and environment-specific paths

Application:
  ports, endpoints and service behavior
```

---

# 4. Configured Public Routing

The earlier development model exposed individual service hostnames.

```text
UI service hostname
user service hostname
product service hostname
order service hostname
```

This remains useful for debugging but is not the preferred final public model.

Current source routing (previously validated; DEV is now OFF):

```text
app.dev.jsapp365.com
  frontend application

api.dev.jsapp365.com/users
  user-service

api.dev.jsapp365.com/products
  product-service

api.dev.jsapp365.com/orders
  order-service
```

A possible future alternative UI hostname is:

```text
ui.dev.jsapp365.com
```

Backend microservices should remain internal implementation details.

---

# 5. Edge Security Architecture

Validated target edge request path:

```text
Internet
  -> Route 53
  -> CloudFront
  -> AWS WAF
  -> Application Load Balancer
  -> Gateway API
  -> HTTPRoute
  -> Kubernetes Service
```

The infrastructure and cutover workflow were validated.

Long-running edge exposure remains intentionally disabled when it is not required for the development environment.

CloudFront responsibilities:

```text
global public entrypoint
HTTPS at the edge
controlled cache behavior
origin request forwarding
future static-asset optimization
```

AWS WAF responsibilities:

```text
AWS managed rule groups
common web-attack protection
known-bad-input protection
rate-based controls
optional request logging
```

API paths should initially avoid aggressive caching.

Static UI assets may use dedicated cache behaviors later.

---

# 6. GitOps Deployment Architecture

Application deployment is separated from AWS infrastructure provisioning.

```text
Application source commit
        |
        v
GitLab CI
        |
        +--> tests
        +--> Docker image build
        +--> immutable version tag
        +--> push image to Amazon ECR
                         |
                         v
                 GitOps repository
                         |
                         v
                      ArgoCD
                         |
                         v
                   Helm rendering
                         |
                         v
       Deployment + Service + HTTPRoute
                         |
                         v
                Kubernetes application Pods
```

ArgoCD reconciles Kubernetes desired state.

Terraform does not manage normal application Deployments.

Source or artifact readiness, image publication, GitOps version selection and live promotion are separate evidence states. The committed release-preparation record retains historical pre-lockdown image selections and does not prove that newer images exist or were promoted. DEV is OFF; this document claims no currently running application versions.

---

# 7. Secrets Architecture

Runtime secrets use this flow:

```text
Terraform infrastructure
        |
        v
RDS application-provisioner Lambda
        |
        +--> generate service database credentials
        |
        +--> store service secrets
        v
AWS Secrets Manager
        |
        v
External Secrets Operator
        |
        v
Kubernetes Secret
        |
        v
application Pod
```

Secret values are not stored directly in GitOps manifests.

Examples:

```text
service-specific database credentials
RabbitMQ credentials
application secrets
repository credentials
```

Validated database secret pattern:

```text
jsappinf/dev/app/db/user-service
jsappinf/dev/app/db/product-service
jsappinf/dev/app/db/order-service
jsappinf/dev/app/db/ui-service
```

The current provisioning Lambda creates or updates the initial database credentials.

Automated credential rotation remains planned.

The planned rotation workflow may use a dedicated Lambda or scheduled job that:

```text
generates a new database password
updates the PostgreSQL role
updates the matching Secrets Manager secret
waits for External Secrets synchronization
validates application reconnection
supports rollback on failure
records rotation status and evidence
```

Public Cognito configuration is not classified as secret.

Examples:

```text
User Pool ID
app client ID
issuer URL
JWKS URL
managed login domain
OAuth scopes
```

---

# 8. Database Architecture

The application uses Amazon RDS for PostgreSQL.

D01 source implementation, canonical integration and local validation are complete; live runtime acceptance has not yet been performed. The source requires verified PostgreSQL TLS: the reviewed RDS CA bundle is distributed through Secrets Manager and External Secrets into a separate mounted CA Secret, Node clients verify the server certificate, Lambda clients verify certificates and hostnames, and lifecycle Jobs use `sslmode=verify-full`. Missing or invalid CA trust fails closed.

Logical application separation:

```text
appdb
  |
  +--> users schema
  |
  +--> products schema
  |
  +--> orders schema
  |
  +--> ui schema
```

Service-specific database users have dedicated schema permissions.

The `ui` schema and `ui_user` database identity are provisioned by the Lambda as part of the standardized service database contract.

The UI session table `ui.http_sessions` and its indexes are implemented in source and included in C01 migrations. There is no UI fixture seed.

Database responsibilities are separated.

The RDS application-provisioner Lambda creates or reconciles:

```text
service database users
service schemas
schema permissions
service credentials
Secrets Manager entries
```

Application migration Jobs own tables, indexes and order data structures. Separate DEV seed Jobs insert fixture users and catalog products; there is no order or UI fixture seeding.

C01 implements immutable, checksum-identified migrations and DEV seeds generated deterministically from JSAPP SQL. GitOps runs migrations, DEV fixture seeds and completion markers in order; each workload has a PreSync gate for the exact lifecycle bundle. These gates require a full Application sync; selective sync skips hooks. No current live DEV execution is claimed.

This separation keeps database identity and privilege provisioning distinct from application schema and seed-data lifecycle.

Validated database identity provisioning:

```text
user-service
product-service
order-service
ui-service
```

Validated application database flows:

```text
user-service
product-service
order-service
```

The UI database identity belongs to the provisioner; its session table belongs to application migrations. Source implementation does not establish current live DEV execution.

---

# 9. Cognito Authentication Architecture

Amazon Cognito is deployed through a separate persistent Terraform stack.

Current infrastructure:

```text
User Pool
public UI app client
managed login domain
resource server
admin group
operator group
customer group
custom API scopes
```

Validated login flow:

```text
Browser
  -> Authorization Code + PKCE
  -> Cognito managed login
  -> authorization code
  -> token exchange
  -> ID token
  -> access token
  -> refresh token
```

JWT verification against Cognito JWKS validated:

```text
RS256 signature
issuer
expiration
token_use
audience or client_id
groups
custom scopes
```

Application integration implemented in source (current-source live acceptance pending):

```text
UI login and callback
token lifecycle
backend JWT middleware
scope authorization
group authorization
Helm public configuration
GitOps environment values
```

Identity and application-profile responsibilities are intentionally separated.

```text
Amazon Cognito:
  user identity
  authentication
  password and MFA policies
  federation
  OAuth clients
  token issuance
  groups and scopes

JSAPP backend services:
  JWT signature and claim validation
  scope and group authorization
  business authorization
  current-user context

user-service:
  application profile
  display and business attributes
  preferences
  application-specific user state
```

The application profile should be linked to the Cognito identity through the stable token subject claim:

```text
Cognito claim:
sub

application profile:
identity_provider = cognito
identity_subject = <Cognito sub>
```

Email should not be the primary identity key because it may change.

A separate local LDAP or custom token-issuing authentication service is not part of the current target architecture.

External corporate identity may be integrated later through Cognito federation using OIDC or SAML without changing the application token-validation boundary.

B03 session/request hardening is implemented in source, including origin checks, explicit proxy trust and process-local session invalidation. B03b cross-replica revocation remains deferred: concurrent refresh/logout across replicas or a failed session deletion can leave credentials usable elsewhere until expiry or invalidation.

---

# 10. RabbitMQ Messaging Architecture

RabbitMQ provides the asynchronous messaging capability used by the application.

Validated application flow:

```text
order-service
      |
      | publishes order.created
      v
RabbitMQ exchange
      |
      v
product-service.order-created queue
      |
      v
product-service consumer
```

Validated runtime behavior:

```text
RabbitMQ connection established
RabbitMQ publisher initialized
RabbitMQ consumer started
order.created event published
order.created event consumed successfully
end-to-end messaging flow verified
```

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
event schemas
publisher logic
consumer logic
acknowledgement behavior
retry behavior
dead-letter handling
idempotency
business processing
```

Current validated application ownership:

```text
order-service:
publisher of order.created events

product-service:
consumer of the product-service.order-created queue
```

The core publisher-to-consumer application flow is implemented and validated.

That runtime validation is historical. Order event publication is best effort after the database commit, and the product consumer validates/logs/acknowledges the event without updating stock. This is not a transactional outbox or a durable business-processing guarantee.

Future enhancements include:

```text
additional consumers
additional business events
formal retry policies
dead-letter queues
message replay procedures
broader idempotency validation
failure and recovery testing
```

---

# 11. Capacity Architecture

When DEV runtime is enabled, the baseline application runs on managed EKS nodes.

Karpenter provides optional dynamic capacity.

Target placement model:

```text
stable replicas:
ON_DEMAND capacity

burst replicas:
SPOT capacity
```

Workload labels and NodePool constraints control placement.

Karpenter remains optional in the cost-aware development environment.

---

# 12. Monitoring and Operations

Platform observability includes or plans:

```text
Prometheus
Grafana
CloudWatch logs
VPC flow logs
AWS WAF metrics
ALB metrics
ArgoCD application health
Kubernetes probes
application health endpoints
```

Operational validation includes:

```text
service health checks
database connectivity
Gateway API routing
GitOps synchronization
autoscaling behavior
secret synchronization
Terraform plan and destroy safety
```

---

# 13. Repository Boundaries

```text
terraform-modules
  reusable AWS Terraform modules

jsappinf-platform
  Terraform root stacks, component registry, shared modules and standards

JSAPP
  Node.js services, Dockerfiles, CI and application integration

helmchartsappjs
  reusable Kubernetes workload packaging

jsappinf-gitops
  environment-specific runtime desired state

jsappinf-platform-portfolio
  sanitized public architecture documentation
```

Related documents:

```text
docs/component-responsibility-matrix.md
docs/repository-ownership-model.md
```

---

# 14. Cost-Aware Lifecycle

The project distinguishes three lifecycle classes.

## Persistent

```text
Cognito
IAM
Route 53
ACM
small S3 state objects
selected public configuration
```

## Pausable

```text
test users
Lambda functions
EventBridge rules
queues
topics
```

## Ephemeral

```text
EKS
managed nodes
NAT Gateway
RDS
ALB
VPC interface endpoints
RabbitMQ runtime
CloudFront and WAF development layer
```

Lifecycle classification does not change Terraform state ownership.

A resource remains owned by exactly one Terraform state.

---

# 15. Current Platform Direction

Current validated direction:

```text
Gateway API instead of ingress-nginx
ALB instead of the earlier NLB ingress path
RabbitMQ asynchronous application messaging
explicit component ownership
separate persistent and ephemeral Terraform states
```

Remaining evolution:

```text
long-running CloudFront and AWS WAF edge exposure
live acceptance of current Cognito UI/backend integration and session hardening
additional RabbitMQ events and consumers
formal retry and dead-letter queue policies
automated database credential rotation
rotation validation and rollback workflow
default-privilege standardization for future application objects
```

---

# 16. Environment and Account Growth Strategy

The current implementation is a close-to-production development platform implementation.

It intentionally demonstrates production-like patterns while remaining practical for a cost-aware AWS development environment:

```text
modular Terraform
remote state
explicit resource ownership
private networking
managed Kubernetes
GitOps reconciliation
identity separation
edge protection
secure secret delivery
cost-aware lifecycle controls
```

The current runtime is not yet a full multi-account platform.

The target growth direction is:

```text
foundation account or foundation layer
shared governance and security controls
separate workload accounts
separate dev, stage and production environments
environment-specific Terraform states
environment-specific Cognito User Pools
controlled DNS and certificate ownership
centralized audit and security visibility
```

Conceptual target:

```text
AWS Organization
  |
  +--> Management / governance
  |
  +--> Security and logging
  |
  +--> Shared services or foundation
  |
  +--> Development workload account
  |
  +--> Staging workload account
  |
  +--> Production workload account
```

The portfolio therefore distinguishes between:

```text
current validated development architecture
and
planned multi-account operating model
```

The current dev implementation is not presented as already having full enterprise governance.

Detailed strategy:

```text
docs/foundation-and-lightweight-landing-zone-strategy.md
docs/multi-env-and-account-strategy.md
```

---

# 17. Cross-Cloud Evolution Strategy

The longer-term project direction is to evolve JSAPPINF into a cross-cloud development-platform reference architecture.

The objective is not to copy AWS Terraform resources directly into Microsoft Azure or Google Cloud.

The objective is to preserve shared platform capabilities, contracts and architecture principles while implementing provider-specific infrastructure.

Portable layers include:

```text
application containers
HTTP API contracts
event contracts
Helm workload packaging
ArgoCD GitOps conventions
Kubernetes workload resources
JWT authorization contracts
health-check conventions
metrics and observability conventions
```

Provider-specific layers include:

```text
networking
cloud IAM
workload identity
managed Kubernetes integration
managed PostgreSQL
load balancing
DNS
certificate management
edge protection
secret stores
key management
organization and account hierarchy
```

AWS remains the first reference implementation, with current-source live acceptance still pending.

Azure and Google Cloud remain planned development-platform equivalents and are not presented as implemented.

The detailed strategy documents:

```text
cloud-native capability equivalence
portable open-source alternatives
hybrid implementation options
cross-cloud identity approaches
portability boundaries
future repository structure
phased Azure and Google Cloud implementation
validation and comparison criteria
```

Detailed document:

```text
docs/cross-cloud-platform-equivalence-strategy.md
```

---

# 18. Portfolio Accuracy

Portfolio documentation distinguishes between:

```text
implemented and validated
infrastructure validated
prepared
planned
previous iteration
```

This prevents planned application integration from being presented as already complete.
