# LinkedIn Featured Description

## Title

JSAPPINF — Three-Environment AWS EKS DevOps Platform

## Description

A production-like DEV/STAGE/PROD platform combining Terraform, AWS EKS, GitLab CI, Helm and Argo CD. Separate repositories own infrastructure, application code, packaging and environment desired state.

The platform includes PostgreSQL provisioning, RabbitMQ, Cognito integration, Secrets Manager with External Secrets Operator, Gateway API/ALB routing, ACM TLS and Prometheus/Grafana monitoring. Shared application code and charts support environment-specific configuration and immutable image promotion.

An end-to-end infrastructure and GitOps run covered startup, deployment, operational fixes and controlled teardown. Manual runbooks and guarded lifecycle scripts capture the process, including independent state bootstrap, protected identity and recovery resources, and cost-aware DEV retirement.

This public repository presents the architecture, ownership model and operational lessons without credentials or private state.
