# Secrets flow

Terraform provisions secret containers, IAM/KMS access and provisioning functions. The database provisioner reconciles service users, schemas and grants and publishes credentials to Secrets Manager. Application migrations own tables and application data.

External Secrets Operator reads approved secret paths through IAM/IRSA and materializes Kubernetes Secrets. Shared charts consume these references; GitOps supplies the environment configuration. Secret values stay out of repositories and public documentation.

Database clients require certificate verification with an explicitly mounted CA bundle. The October run confirmed successful provisioning and stable current secret versions across a repeated invocation.

See [the architecture](platform-architecture.md#7-secrets-architecture) and [operations](platform-operations.md) for ownership and lifecycle details.
