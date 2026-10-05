# DEV, STAGE and PROD configuration model

JSAPPINF is a three-environment platform built from shared infrastructure composition, application source and Helm charts. Environment differences belong in explicit inputs and GitOps configuration.

| Concern | Shared implementation | Environment selection |
|---|---|---|
| AWS platform | `stacks/aws/platform` | `env/aws/private/<env>.tfvars` |
| State bootstrap | `bootstrap/aws-state` | Dedicated environment bucket, encryption and locking configuration |
| Backend | S3 backend contract | Environment-specific backend configuration and state keys |
| Identity | Cognito module; `stacks/aws/identity` | Environment identity inputs and independent state |
| Application | One JSAPP codebase | Promoted image version/digest |
| Packaging | One chart per service | Environment values and configuration |
| GitOps | Shared conventions and chart contracts | `environments/dev`, `environments/stage`, `environments/prod` |

## State and identity isolation

Environment separation uses explicit backend configuration, not an additional Terraform workspace hierarchy. The `default` workspace is used with independently selected backend state. AWS account, region, environment and backend are checked together before operations.

Each environment has its own backend bootstrap configuration. Native S3 lockfiles and provider dependency lockfiles serve different purposes: the former coordinate state writes, while `.terraform.lock.hcl` pins provider selections.

The existing DEV backend key is retained for continuity even though it contains the historical `infra-next` name. Moving local directories does not rename remote state. Existing DEV Cognito also retains its independent legacy identity root/state; new environment identity uses the shared identity root. No implicit state migration is performed.

## Environment profiles

- **DEV:** frequent rebuilds, cost-aware capacity and explicit disposable-resource cleanup.
- **STAGE:** integration configuration, controlled promotion and protected recovery resources.
- **PROD:** conservative changes, protected database lifecycle and deliberate retirement approval.

Changing only an environment label is insufficient. Inputs also bind names, domains, network configuration, access restrictions, secrets, image repositories and identity clients. Multiple environments can use separate VPCs in one account; shared subnet ownership must be designed explicitly. Future account separation can provide a stronger isolation boundary.

## GitOps and Helm

Charts define reusable configuration contracts. GitOps provides environment-specific trusted proxy networks, identity issuer/client settings, approved customer links, routing, policies and promotion references.

DEV inherits service image defaults from charts. STAGE/PROD may pin image digests as explicit promotion decisions. They do not need copies of application source or chart templates. Kustomize composes environment resources; Argo CD consumes those resources and Helm values through its Applications.

## Cognito ownership

Terraform owns pools, clients, resource servers and infrastructure-level identity configuration. User administration remains a separate operation. GitOps owns references and approved application mappings, never passwords. Application code validates tokens and resolves customer identity into its own database model.

## Account evolution

AWS Organizations, a lightweight landing zone and dedicated production accounts remain optional future governance work. They are not prerequisites for the three-environment configuration model.

See [Platform operations](platform-operations.md) for startup and retirement order.
