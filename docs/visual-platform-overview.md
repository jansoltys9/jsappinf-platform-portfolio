# JSAPP Platform Visual Overview

This page provides a visual summary of the JSAPP platform architecture, repository model, delivery process, ownership boundaries and application landscape.

The diagrams combine validated platform capabilities with clearly identified target-state and work-in-progress areas.

## Infrastructure Overview

![JSAPP platform infrastructure overview](../diagrams/platform-overview/jsapp-platform-infrastructure-overview.png)

### Status

Validated foundation:

- AWS VPC and subnet design
- Amazon EKS and managed node groups
- Karpenter-based capacity model
- Amazon RDS PostgreSQL
- Amazon MQ RabbitMQ
- Amazon ECR
- AWS Secrets Manager
- AWS Load Balancer Controller
- ExternalDNS
- External Secrets
- cert-manager
- Terraform, Helm and Argo CD delivery boundaries

Target edge direction:

```text
Internet
→ Route 53
→ CloudFront
→ AWS WAF
→ Application Load Balancer
→ Gateway API
→ HTTPRoute
→ Kubernetes Service
```

Long-running CloudFront and AWS WAF exposure remains intentionally disabled in the development environment.

## Repository Landscape

![JSAPP platform repository landscape](../diagrams/platform-overview/jsapp-platform-repository-landscape.png)

### Status

Current repository ownership model:

- `JSAPP` owns application source code and service runtime behavior.
- `helmchartsappjs` owns reusable Helm packaging.
- `jsappinf-gitops` owns Argo CD desired state and environment delivery.
- `jsappinf-platform` owns environment composition and AWS platform infrastructure.
- `terraform-modules` owns reusable and versioned infrastructure modules.

This separation reduces overlap and keeps application, packaging, GitOps and infrastructure responsibilities explicit.

## Deployment Process

![JSAPP platform deployment process](../diagrams/platform-overview/jsapp-platform-deployment-process.png)

### Status

Reference end-to-end delivery model:

```text
Source change
→ build and validation
→ versioned artifact
→ GitOps desired-state update
→ Argo CD synchronization
→ Kubernetes runtime
→ service exposure
→ operational validation
```

The exact path depends on the type of change.

Application image changes, Helm chart changes and Terraform platform changes do not always use identical pipelines, but they follow the same principles of versioning, validation and controlled promotion.

## Responsibility Boundaries

![JSAPP platform responsibility boundaries](../diagrams/platform-overview/jsapp-platform-responsibility-boundaries.png)

### Status

Current architectural ownership standard:

- Terraform owns AWS infrastructure, IAM, networking and managed-service prerequisites.
- Helm owns reusable Kubernetes templates and deployment packaging.
- Argo CD and GitOps own declarative environment delivery and synchronization.
- Application services own business logic, APIs, database usage and messaging behavior.

These boundaries support safer changes, clearer troubleshooting and more controlled platform evolution.

## Application Landscape

![JSAPP platform application landscape](../diagrams/platform-overview/jsapp-platform-application-landscape.png)

### Status

Validated application capabilities:

- UI, user, product and order services
- PostgreSQL-backed runtime
- RabbitMQ `order.created` publisher and consumer flow
- secret delivery through External Secrets and AWS Secrets Manager
- Gateway API and ALB routing direction
- platform monitoring integration

Work in progress:

- complete Cognito runtime integration
- end-to-end authenticated user flow
- final public application and API routing model
- additional RabbitMQ events, retries and dead-letter handling

## Summary

The platform is intentionally split into clear layers:

```text
Application code
→ reusable packaging
→ GitOps desired state
→ Kubernetes runtime
→ AWS infrastructure foundation
```

The visual models are designed to explain both the current validated platform and the direction of its ongoing evolution.
