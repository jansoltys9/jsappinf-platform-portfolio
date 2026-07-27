# Cross-Cloud Platform Equivalence Strategy

## Purpose

This document defines the long-term direction for evolving JSAPPINF into a cross-cloud development platform reference architecture.

The objective is not to create identical infrastructure in AWS, Microsoft Azure and Google Cloud.

The objective is to implement comparable platform capabilities while respecting the architecture, operational model and strengths of each cloud provider.

The proposed long-term project identity is:

```text
JSAPPINF Cross-Cloud Dev Platform Reference Architecture
```

---

## Current Position

The current implementation is a close-to-production AWS development platform.

It includes:

```text
Amazon EKS
Terraform
Kubernetes Gateway API
AWS Load Balancer Controller
Application Load Balancer
CloudFront
AWS WAF
Amazon Cognito
RDS PostgreSQL
RabbitMQ application messaging flow
ArgoCD
Helm
External Secrets Operator
Karpenter
Prometheus and Grafana
```

The Azure and Google Cloud implementations are future development-platform equivalents.

They are not currently presented as implemented.

---

# 1. Cross-Cloud Design Principle

The project will compare platform capabilities rather than resource names.

```text
same capability goal
different cloud-native implementation
consistent architecture principles
consistent repository boundaries
consistent GitOps workflow
consistent security expectations
consistent validation criteria
```

The implementation should not assume:

```text
identical APIs
identical Terraform resources
identical networking behavior
identical identity models
identical load-balancer features
identical pricing
identical operational responsibility
```

Cloud-provider services may be approximate equivalents without being functionally identical.

---

# 2. Primary Cloud-Native Equivalence Matrix

| Platform capability | AWS reference implementation | Azure equivalent candidate | Google Cloud equivalent candidate |
|---|---|---|---|
| Organization hierarchy | AWS Organizations | Microsoft Entra tenant and Management Groups | Google Cloud Organization and folders |
| Governance foundation | AWS Control Tower or lightweight foundation | Azure Landing Zone | Google Cloud enterprise foundations |
| Account boundary | AWS account | Azure subscription | Google Cloud project |
| Resource grouping | Tags and resource hierarchy | Resource groups and tags | Projects, folders and labels |
| Infrastructure as Code | Terraform / OpenTofu | Terraform / OpenTofu | Terraform / OpenTofu |
| Virtual network | Amazon VPC | Azure Virtual Network | Google Cloud VPC |
| Private subnets | VPC private subnets | VNet subnets | VPC subnets |
| Network filtering | Security Groups and network ACLs | Network Security Groups | VPC firewall rules and policies |
| Private service access | VPC endpoints / PrivateLink | Private Endpoint / Private Link | Private Service Connect |
| Managed Kubernetes | Amazon EKS | Azure Kubernetes Service | Google Kubernetes Engine |
| Managed worker capacity | EKS managed node groups | AKS node pools | GKE node pools |
| Dynamic Kubernetes capacity | Karpenter | AKS node autoscaling or Karpenter support where appropriate | GKE Cluster Autoscaler / node auto-provisioning |
| Container registry | Amazon ECR | Azure Container Registry | Artifact Registry |
| Managed PostgreSQL | Amazon RDS for PostgreSQL | Azure Database for PostgreSQL | Cloud SQL for PostgreSQL |
| Managed secrets | AWS Secrets Manager | Azure Key Vault | Secret Manager |
| Key management | AWS KMS | Azure Key Vault Managed HSM / keys | Cloud KMS |
| Public DNS | Amazon Route 53 | Azure DNS | Cloud DNS |
| Certificate management | AWS Certificate Manager | Azure-managed certificates / Key Vault certificates | Certificate Manager |
| Regional application load balancing | Application Load Balancer | Application Gateway | Cloud Load Balancing |
| Global edge entrypoint | CloudFront | Azure Front Door | Cloud CDN with global load balancing |
| Web application firewall | AWS WAF | Azure Web Application Firewall | Cloud Armor |
| Customer identity | Amazon Cognito | Microsoft Entra External ID | Identity Platform |
| Workforce identity | IAM Identity Center / external federation | Microsoft Entra ID | Cloud Identity / Workforce Identity Federation |
| Messaging | RabbitMQ, Amazon MQ, SNS/SQS or EventBridge | RabbitMQ, Service Bus or Event Grid | RabbitMQ, Pub/Sub or Eventarc |
| Functions | AWS Lambda | Azure Functions | Cloud Run functions |
| Object storage | Amazon S3 | Azure Blob Storage | Cloud Storage |
| Monitoring | CloudWatch | Azure Monitor | Cloud Monitoring |
| Audit logging | CloudTrail | Azure Activity Log | Cloud Audit Logs |
| Security posture | Security Hub and related services | Microsoft Defender for Cloud | Security Command Center |
| GitOps | ArgoCD | ArgoCD | ArgoCD |

This table represents capability candidates, not guarantees of identical behavior.

Each implementation requires a provider-specific architecture review.

---

# 3. Managed Kubernetes Reference Model

Managed Kubernetes provides the strongest portable runtime boundary.

```text
AWS:
Amazon EKS

Azure:
Azure Kubernetes Service

Google Cloud:
Google Kubernetes Engine
```

The Kubernetes workload layer can retain many common elements:

```text
Deployments
Services
ConfigMaps
Secrets references
probes
HorizontalPodAutoscaler
Gateway API
Helm charts
ArgoCD Applications
Prometheus metrics
OpenTelemetry instrumentation
```

Provider-specific integration remains necessary for:

```text
identity and workload federation
load balancers
persistent storage
DNS
certificate management
networking
node provisioning
secret stores
logging backends
```

The target is therefore:

```text
portable application layer
portable GitOps layer
cloud-specific infrastructure integration layer
```

---

# 4. Portable Open-Source Layer

Some components can remain substantially consistent across all three clouds.

| Capability | Portable open-source candidate | Cloud-native alternative |
|---|---|---|
| Container orchestration | Kubernetes | EKS, AKS or GKE |
| Application packaging | Helm | Provider-specific deployment services |
| GitOps | ArgoCD | Provider-native pipeline and deployment tools |
| Kubernetes ingress/routing | Gateway API with portable controller | Provider-managed ingress and load balancing |
| Identity provider | Keycloak | Cognito, Entra External ID, Identity Platform |
| Secrets abstraction | External Secrets Operator | Native secret-store integrations |
| Central secret system | HashiCorp Vault | Secrets Manager, Key Vault, Secret Manager |
| Messaging | RabbitMQ, Apache Kafka or NATS | Amazon MQ/SQS/SNS, Service Bus/Event Grid, Pub/Sub |
| Metrics | Prometheus | CloudWatch, Azure Monitor, Cloud Monitoring |
| Dashboards | Grafana | Provider-native dashboards |
| Logging | Loki or OpenSearch | Provider-native logging services |
| Tracing | Tempo and OpenTelemetry | Provider-native tracing services |
| Policy enforcement | Open Policy Agent / Gatekeeper or Kyverno | Provider-native policy services |
| Certificate automation | cert-manager | Provider certificate services |
| PostgreSQL on Kubernetes | PostgreSQL operator such as CloudNativePG | Managed PostgreSQL services |
| Infrastructure provisioning | Terraform or OpenTofu | Provider-native IaC systems |
| Kubernetes control-plane provisioning | Crossplane | Terraform or provider-native IaC |

Open-source portability reduces provider coupling but transfers more operational responsibility to the platform team.

Cloud-native services usually reduce operational effort but increase provider-specific integration.

---

# 5. Three Implementation Modes

## Mode A: Cloud-Native Equivalence

Each cloud uses its own managed services.

```text
AWS:
EKS + RDS + Cognito + CloudFront + WAF

Azure:
AKS + Azure Database for PostgreSQL
+ Entra External ID + Front Door + WAF

Google Cloud:
GKE + Cloud SQL + Identity Platform
+ Cloud Load Balancing + Cloud CDN + Cloud Armor
```

Advantages:

```text
strong provider integration
managed-service operations
native IAM integration
native monitoring
native support model
```

Trade-offs:

```text
different Terraform resources
different operational procedures
different identity integration
different networking behavior
greater cloud-provider coupling
```

---

## Mode B: Portable Open-Source Platform

Each cloud provides mainly:

```text
networking
managed Kubernetes
worker compute
persistent storage
DNS
base identity integration
```

The platform provides shared components:

```text
ArgoCD
Helm
Gateway API
Keycloak
RabbitMQ
Prometheus
Grafana
OpenTelemetry
External Secrets Operator
policy engine
```

Advantages:

```text
more consistent runtime model
greater workload portability
shared operational knowledge
reusable Kubernetes manifests
reusable Helm charts
consistent application integration
```

Trade-offs:

```text
higher operational responsibility
upgrades owned by the platform team
backup and recovery ownership
high-availability design
security patching
capacity planning
```

---

## Mode C: Hybrid Platform

The platform intentionally combines services from multiple providers or from an independent open-source layer.

Examples:

```text
central identity in one location
workloads in multiple clouds

central GitOps control plane
multiple Kubernetes target clusters

shared observability backend
telemetry collected from multiple clouds

shared source-control and CI platform
deployments targeting multiple providers

cloud-native databases per provider
portable application and GitOps layers
```

Hybrid architecture should be introduced only where it has a clear operational or business reason.

It must not be adopted merely to claim multi-cloud support.

---

# 6. Hybrid Authentication and Authorization

Identity is a strong candidate for a shared cross-cloud capability because applications can integrate through standard protocols:

```text
OpenID Connect
OAuth 2.0
SAML 2.0
JWT
JWKS
```

The identity provider and the application workload do not have to run in the same cloud.

Conceptual flow:

```text
Central identity provider
        |
        | OIDC / OAuth 2.0
        v
Browser or client application
        |
        | signed access token
        v
AWS, Azure or Google Cloud workload
        |
        +--> verify JWT signature
        +--> verify issuer
        +--> verify audience or client ID
        +--> verify expiration
        +--> verify scopes
        +--> verify groups or roles
```

Possible approaches follow.

---

## Option A: Cognito as the Initial Shared Application Identity

```text
Amazon Cognito
      |
      +--> application on EKS
      |
      +--> future application on AKS
      |
      +--> future application on GKE
```

Applications outside AWS can validate Cognito-issued JWTs through the public OIDC issuer and JWKS endpoints.

Potential advantages:

```text
build on the currently validated Cognito implementation
reuse Authorization Code + PKCE
reuse groups and scopes
one initial identity model
avoid implementing three identity systems immediately
```

Considerations:

```text
identity dependency remains in AWS
cross-cloud network availability must be considered
outage and disaster-recovery boundaries must be documented
provider coupling remains at the identity layer
data residency requirements must be reviewed
```

---

## Option B: Microsoft Entra as Central Enterprise Identity

```text
Microsoft Entra ID
      |
      +--> JSAPP UI and APIs
      |
      +--> AWS workloads
      |
      +--> Azure workloads
      |
      +--> Google Cloud workloads
      |
      +--> GitLab and operational tooling
```

This model is especially relevant for organizations already using Microsoft identity and productivity services.

For an Entra-first internal enterprise application, JSAPP may trust Entra-issued OIDC tokens directly.

Cognito is not required as an intermediate broker unless there is a specific need for migration compatibility, multiple upstream identity providers or a normalized AWS-owned token boundary.

Application-customer identity and workforce identity should still be treated as separate concerns.

---

## Option C: Keycloak as Portable Identity Provider or Broker

```text
Local or external identities
      |
      | OIDC, SAML or user federation
      v
Keycloak
      |
      +--> EKS applications
      +--> AKS applications
      +--> GKE applications
```

Keycloak may run on cloud-neutral compute such as:

```text
Amazon EC2
Azure Virtual Machines
Google Compute Engine
managed Kubernetes in any of the three clouds
```

The deployment model must still address high availability, database persistence, backup, recovery and upgrades.

Keycloak can provide:

```text
OpenID Connect
OAuth 2.0
SAML
identity brokering
user federation
central client configuration
roles and authorization policies
```

Potential advantages:

```text
cloud-neutral identity layer
portable configuration model
integration with external identity providers
consistent tokens for applications
```

Considerations:

```text
platform team owns operation
high availability
database persistence
backup and restore
security upgrades
key rotation
monitoring
disaster recovery
```

---

## Option D: Federation or Brokering Where Justified

A hybrid model may retain provider-specific or corporate identity systems and connect them through federation.

An additional broker should be introduced only when it solves a concrete requirement, such as:

```text
multiple upstream identity providers
local and corporate users in one application
incremental migration between identity platforms
claim normalization across providers
a stable issuer contract during migration
```

For one enterprise identity provider and one application boundary, direct OIDC integration may be simpler and clearer.

Example:

```text
Microsoft Entra ID
        |
        | OIDC or SAML federation
        v
Amazon Cognito
        |
        | normalized OIDC tokens
        v
JSAPP applications
```

Alternative:

```text
corporate identity providers
        |
        v
Keycloak identity broker
        |
        v
applications across multiple clouds
```

The application then trusts one defined issuer instead of implementing every upstream identity provider separately.

---

# 7. Authentication Versus Cloud Authorization

Application authentication must remain separate from infrastructure authorization.

## Application identity

Examples:

```text
customer login
operator login
admin login
OAuth scopes
application roles
API access tokens
```

Possible providers:

```text
Cognito
Entra External ID
Identity Platform
Keycloak
```

## Cloud infrastructure identity

Examples:

```text
Pod accessing a secret
CI pipeline pushing an image
Terraform assuming a deployment role
workload accessing cloud APIs
operator accessing a cluster
```

Provider-specific mechanisms include:

```text
AWS IAM and workload roles
Azure managed identities and workload identity
Google Cloud IAM and Workload Identity Federation
```

A user access token must not automatically become infrastructure credentials.

---

# 8. Recommended JSAPPINF Evolution

## Phase 1: AWS Reference Implementation

```text
complete Cognito application integration
expand RabbitMQ events, consumers and reliability controls
finalize app/API routing
validate long-running CloudFront and WAF runtime path
stabilize EKS platform baseline
```

## Phase 2: Cloud-Independent Contract

Define provider-neutral contracts for:

```text
application configuration
identity claims
OAuth scopes
group and role mapping
secret names
database connectivity
event schemas
health endpoints
observability
GitOps promotion
```

## Phase 3: Azure Development Equivalent

Potential initial scope:

```text
Azure Virtual Network
AKS
Azure Container Registry
Azure Database for PostgreSQL
Key Vault
Application Gateway or Front Door
Azure WAF
Azure DNS
ArgoCD
existing JSAPP Helm charts
```

Identity decision:

```text
use Entra directly for an Entra-first enterprise scenario
or
reuse Cognito temporarily during migration
or
use Keycloak as a shared portable identity provider
```

## Phase 4: Google Cloud Development Equivalent

Potential initial scope:

```text
Google Cloud VPC
GKE
Artifact Registry
Cloud SQL for PostgreSQL
Secret Manager
Cloud Load Balancing
Cloud CDN
Cloud Armor
Cloud DNS
ArgoCD
existing JSAPP Helm charts
```

Identity decision:

```text
reuse the selected shared identity
or
implement Google Cloud Identity Platform
```

## Phase 5: Cross-Cloud Validation

Compare:

```text
Terraform structure
deployment time
operational complexity
networking model
identity integration
secret delivery
load-balancer behavior
observability
cost
destroy and rebuild behavior
application portability
```

---

# 9. Portability Boundaries

## Highly portable

```text
application containers
business logic
HTTP APIs
event schemas
Helm chart concepts
Kubernetes Deployments and Services
probes
resource requests and limits
ArgoCD model
Prometheus metrics
OpenTelemetry instrumentation
JWT validation logic
```

## Partially portable

```text
Gateway API configuration
persistent-volume configuration
autoscaling
secret-store integration
DNS automation
certificate automation
workload identity
monitoring integration
```

## Provider-specific

```text
VPC and subnet implementation
IAM resources
load-balancer annotations and policies
managed database resources
edge services
WAF rules
cloud organization structure
billing and quotas
private service connectivity
KMS integration
```

---

# 10. Repository Model for Cross-Cloud Growth

Possible future structure:

```text
terraform-modules
  |
  +--> aws
  +--> azure
  +--> gcp
  +--> shared conventions

jsappinf-platform
  |
  +--> envs/aws/dev
  +--> envs/azure/dev
  +--> envs/gcp/dev
  +--> modules/aws
  +--> modules/azure
  +--> modules/gcp
  +--> standards

helmchartsappjs
  portable application packaging

jsappinf-gitops
  |
  +--> environments/aws/dev
  +--> environments/azure/dev
  +--> environments/gcp/dev

JSAPP
  portable application source and container images

jsappinf-platform-portfolio
  sanitized public architecture and comparison documentation
```

The exact repository split should be decided only after the first Azure implementation demonstrates where real duplication exists.

Premature universal abstraction should be avoided.

---

# 11. Cross-Cloud Responsibility Matrix

| Concern | Shared responsibility | Cloud-specific responsibility |
|---|---|---|
| Application source | `JSAPP` | provider-specific SDK use only where necessary |
| Containers | common Dockerfiles | target registry and image identity |
| Helm charts | reusable workload shape | provider-specific values |
| GitOps | common ArgoCD conventions | cluster destinations and environment values |
| Terraform standards | naming, tags, module interfaces | provider resources and IAM |
| Identity contract | claims, scopes and roles | selected IdP and federation configuration |
| Secrets contract | application secret keys | provider secret store |
| Database contract | PostgreSQL schemas and migrations | managed database implementation |
| Messaging contract | event schemas | broker or managed service |
| Networking contract | required flows | VPC, VNet or GCP VPC implementation |
| Edge contract | public routes and security expectations | CloudFront, Front Door or Cloud CDN |
| Observability | metrics and tracing conventions | provider export and storage |
| Validation | common test scenarios | provider-specific evidence |

---

# 12. Architecture Decision Guidelines

Choose cloud-native services when:

```text
reduced operational work is important
provider integration provides clear value
the service is difficult to operate safely
strong managed availability is required
```

Choose open-source services when:

```text
consistent behavior across clouds is valuable
the team can operate the component safely
provider portability has measurable value
the operational burden is acceptable
```

Choose hybrid architecture when:

```text
there is an existing central identity platform
regulation requires workload separation
business continuity requires provider diversity
a shared control plane reduces duplication
migration must happen incrementally
```

Do not choose hybrid architecture when:

```text
it only increases presentation value
the team cannot operate the added complexity
cross-cloud traffic creates unnecessary latency
egress costs outweigh the benefit
failure ownership becomes unclear
```

---

# 13. Initial Recommended Identity Position

For JSAPPINF, the practical initial direction is:

```text
AWS remains the first complete reference implementation.

Cognito remains the validated identity provider
for the AWS reference implementation.

Kubernetes, Helm, ArgoCD, application containers,
JWT validation contracts and authorization conventions
form the portable application and platform layer.

The application should keep issuer, audience, JWKS,
scope and role-claim mapping configurable.

An Entra-first enterprise implementation may trust
Entra-issued OIDC tokens directly without Cognito.

Keycloak remains a candidate for a shared cloud-neutral
identity-provider or identity-broker implementation.

Cognito brokering for Entra or another external provider
remains optional and requires a concrete architectural reason.

Managed PostgreSQL and networking remain cloud-native.
```

This preserves application portability without forcing one identity provider or an unnecessary broker layer across all three clouds.

---

# 14. Status

## Implemented

```text
AWS development-platform reference implementation
Kubernetes application packaging
ArgoCD GitOps model
Cognito infrastructure and token validation
portable Node.js application containers
RabbitMQ order.created publisher-to-consumer flow
RDS application-provisioner Lambda
service database users, schemas and grants
service credential generation and Secrets Manager storage
External Secrets delivery into Kubernetes
```

## Planned

```text
Cognito integration into the AWS reference application
provider-neutral JWT and authorization contracts
configurable issuer, audience, JWKS and claim mapping
automated database credential rotation
rotation validation and rollback workflow
additional RabbitMQ events and consumers
formal retry and dead-letter queue policies
Azure development equivalent
Google Cloud development equivalent
direct Entra identity evaluation
Keycloak deployment and operations comparison
optional identity-broker evaluation
cross-cloud operational comparison
```

No Azure or Google Cloud runtime is currently presented as implemented.

---

# 15. Related Documentation

```text
docs/platform-architecture.md
docs/component-responsibility-matrix.md
docs/multi-env-and-account-strategy.md
docs/foundation-and-lightweight-landing-zone-strategy.md
docs/repository-ownership-model.md
```
