# JSAPPINF — Three-Environment AWS EKS Platform

JSAPPINF is a production-like DevOps platform for **DEV, STAGE and PROD**, built with Terraform, AWS EKS, GitLab CI, Helm and Argo CD. It connects infrastructure provisioning, application delivery, database provisioning, secrets management and controlled environment retirement into one documented operating model.

This public repository presents the architecture and engineering decisions. Implementation repositories remain private in GitLab; credentials, Terraform state and private environment inputs are excluded.

## Current platform

One Terraform composition serves three environment profiles. Each environment selects its own inputs, backend, resource names, secret paths and GitOps configuration. Application source and Helm charts are shared across environments.

The October 2, 2026 operational run exercised the shared platform root from infrastructure startup through GitOps deployment and staged teardown. EKS nodes and platform workloads became ready, database provisioning completed, application images reached the cluster, HTTPS routing worked and monitoring was available. The platform was then retired in dependency order for cost control, retaining protected state, identity and recovery resources.

The application provides a realistic platform workload: UI, user, product and order services with PostgreSQL and RabbitMQ. Further work focuses on the complete customer journey and local development tooling.

## Architecture

- **Infrastructure:** VPC, private EKS worker nodes, ECR, RDS PostgreSQL, Amazon MQ for RabbitMQ, IAM/IRSA and KMS.
- **Routing:** Gateway API and AWS Load Balancer Controller provision ALB routing; ACM provides listener certificates and ExternalDNS manages DNS records.
- **Secrets:** AWS Secrets Manager → External Secrets Operator → Kubernetes Secrets. Database clients verify TLS using an explicit CA bundle.
- **Identity:** Terraform owns Cognito infrastructure. Application code owns authentication and customer-link behavior; GitOps supplies environment-specific identity configuration.
- **Delivery:** GitLab CI builds versioned service images; Helm defines workload structure; Argo CD reconciles environment desired state.
- **Observability:** Prometheus, Grafana and Alertmanager are deployed through GitOps.
- **Optional capabilities:** Karpenter capacity profiles and CloudFront/WAF edge configuration.

The former ingress-nginx/NLB path is historical. The current ALB/ACM routing path does not require cert-manager; it remains an optional component for other certificate use cases.

## Repository ownership

| Repository | Responsibility |
|---|---|
| `terraform-modules` | Reusable infrastructure modules and versioned contracts |
| `jsappinf-platform` | Environment composition, backend bootstrap, Cognito infrastructure, platform add-ons, provisioning Lambdas and lifecycle operations |
| `JSAPP` | Four Node.js services, Dockerfiles, tests, database migrations, seed definitions and CI release logic |
| `helmchartsappjs` | Shared charts, configuration schemas, deployments, probes and service contracts |
| `jsappinf-gitops` | DEV/STAGE/PROD desired state, routes, policies, secret references, migration orchestration and application configuration |
| `jsappinf-platform-portfolio` | Sanitized public architecture and operational documentation |

A resource or configuration decision has one owner. Terraform does not manage application tables; GitOps does not create Cognito users or own passwords; charts do not hard-code environment-specific identity mappings or trusted proxy networks.

## Startup and shutdown

Startup proceeds through backend bootstrap and identity preparation, followed by three platform phases:

1. **Foundation:** Terraform with cluster runtime disabled creates the AWS resources.
2. **Provisioning:** Lambda reconciles database users, schemas, grants and credentials.
3. **Runtime:** Terraform installs cluster-dependent components; GitOps then delivers migrations, monitoring and application workloads.

Shutdown reverses dependencies: stop reconciliation, remove GitOps workloads and controller-managed load balancers, remove cluster runtime, then retire services, EKS and networking. Protected backend, identity and recovery resources have separate lifecycles. STAGE/PROD retirement adds explicit approval records and database recovery safeguards.

See [Platform operations](docs/platform-operations.md) for the readable workflow and private runbook locations.

## October engineering findings

- Reconciled private DEV inputs with the shared Terraform root, including tag contracts and restricted API access.
- Confirmed repeat database provisioning retained the same current secret versions.
- Updated dependencies and TLS test fixtures across all four services; local service checks and production dependency audits passed.
- Removed duplicate DEV image overrides so chart defaults have one owner; environment promotion can pin immutable digests.
- Fixed trusted proxy configuration through GitOps, restoring secure UI session cookie issuance behind the ALB.
- Added explicit approved customer identity mapping without moving Cognito infrastructure ownership into GitOps.
- Completed GitOps cleanup before removing Terraform-managed controllers and networking.
- Consolidated related Terraform files without changing resource addresses; archived legacy `infra-next` locally while preserving private inputs and state evidence.

## Documentation

- [Current platform status](docs/platform-status.md)
- [Platform operations](docs/platform-operations.md)
- [DEV / STAGE / PROD configuration model](docs/multi-env-and-account-strategy.md)
- [Visual overview](docs/visual-platform-overview.md)
- [Detailed platform architecture](docs/platform-architecture.md)
- [Repository ownership](docs/repository-ownership-model.md)
- [Component responsibility matrix](docs/component-responsibility-matrix.md)
- [GitOps delivery flow](docs/gitops-deployment-flow.md)
- [Secrets flow](docs/secrets-flow.md)
- [CloudFront and WAF](docs/edge-security-cloudfront-waf.md)
- [Optional Karpenter capacity](docs/karpenter-capacity-flow.md)
- [Future account governance](docs/foundation-and-lightweight-landing-zone-strategy.md)
- [Cross-cloud capability mapping](docs/cross-cloud-platform-equivalence-strategy.md)

## Next engineering work

Complete the customer-facing application flow, add local Docker Compose development, review ALB-to-workload security boundaries and extend edge/recovery testing. AWS account governance and cross-cloud implementations remain separate future directions.
