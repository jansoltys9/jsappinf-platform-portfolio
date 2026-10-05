# Platform operations

## First environment startup

1. Select environment, account and region; prepare complete private inputs and backend bindings.
2. Bootstrap the environment state bucket, encryption and locking. Keep bootstrap state protected and independent of workload retirement.
3. Initialize the identity root against its own state and create the Cognito configuration. Preserve the existing DEV identity ownership when rebuilding DEV.
4. Initialize `stacks/aws/platform` with the selected backend and private inputs; review the foundation plan with `enable_cluster_runtime=false`.
5. Apply foundation, build the provisioning Lambda artifacts as required and run database provisioning. Verify the function result and secret version metadata.
6. Configure cluster access and review/apply the runtime plan with `enable_cluster_runtime=true`.
7. Bootstrap Argo CD repository access and the selected GitOps root. Reconcile migrations, readiness gates, workloads and monitoring.
8. Verify rollout status, image references, secret delivery, HTTPS routes and application behavior.

The operational runbook contains exact commands; this public guide intentionally omits private identifiers and backend values.

## Application release

Run service validation, dependency security checks and the dependency freshness gate. Keep package/lock versions consistent, publish a new immutable service tag and verify image availability. Update the chart or environment promotion reference, render with the intended GitOps values, then publish the desired-state change. Argo CD performs the rollout.

Reusing an existing release tag hides the relationship between code and image; the release workflow uses new versions instead.

## Controlled shutdown

1. Stop automated reconciliation and remove the root Application safely.
2. Delete Gateway resources while the load balancer controller is running; confirm load balancers and target bindings are gone.
3. Cascade workload and platform Applications in dependency order. Check persistent storage and finalizers before removing controllers.
4. Apply the Terraform runtime-off phase while the Kubernetes API remains reachable.
5. Retire service infrastructure, then EKS, then networking, then the permitted cleanup resources.
6. Verify remaining state and resources. Preserve backend encryption keys, identity and required recovery material.

The Friday teardown used reviewed targeted Terraform phases. Targeting is a controlled lifecycle tool here, not a substitute for reviewing the full remaining state. Low parallelism alone does not define dependency order.

DEV disposal and STAGE/PROD retirement have different policies. Protected environment retirement requires explicit approval metadata, database snapshot safeguards and separate review of deletion protection. Scripts do not silently purge secrets, destroy backend keys or remove finalizers to force progress.

## Implementation runbooks

| Repository | Document |
|---|---|
| `jsappinf-platform` | `docs/operations/FULL_MANUAL_RUNBOOK.md` |
| `jsappinf-platform` | `docs/operations/OPERATIONS_QUICKSTART.md` |
| `jsappinf-platform` | `docs/operations/SAFE_SHUTDOWN.md` |
| `jsappinf-gitops` | `docs/operations/ENVIRONMENT_STARTUP.md` |
| `jsappinf-gitops` | `docs/operations/APP_RELEASE_RUNBOOK.md` |

Terraform owns infrastructure; Argo CD owns its runtime resources. Removing GitOps-created AWS dependencies before Terraform controllers preserves that ownership during shutdown.
