# LinkedIn Featured Description

JSAPPINF – AWS EKS DevOps Platform Lab

Production-like DevOps platform project built on AWS EKS with Terraform, Kubernetes, Helm, ArgoCD GitOps, External Secrets Operator, AWS Secrets Manager, RDS PostgreSQL, ECR, Route 53, cert-manager, and ingress-nginx.

The project demonstrates end-to-end platform engineering practices: infrastructure as code, GitOps-based application delivery, secure secrets integration, Kubernetes ingress and TLS, immutable container image delivery, service-to-service communication, and operational troubleshooting.

The application layer includes multiple Node.js microservices: UI, user, product, and order services. Application images are built through GitLab CI, pushed to AWS ECR, packaged through Helm charts, and deployed to EKS through ArgoCD.

The platform is split across dedicated repositories for reusable Terraform modules, environment infrastructure, application source code, Helm charts, and GitOps runtime state. This separation reflects a production-like ownership model between infrastructure provisioning, application build, Kubernetes packaging, and runtime deployment.

Secrets are not stored directly in GitOps manifests. Runtime secrets are managed through AWS Secrets Manager and synchronized into Kubernetes using External Secrets Operator.

Recent platform work includes a documented CloudFront + AWS WAF edge security design, component registry integration, and Terraform skeleton for adding an edge protection layer in front of the existing NLB + ingress-nginx architecture.

Focus areas:
- AWS EKS platform design
- Terraform infrastructure provisioning
- GitOps with ArgoCD
- Helm-based microservice deployment
- GitLab CI/CD and AWS ECR image delivery
- Secrets management with AWS Secrets Manager and External Secrets Operator
- RDS PostgreSQL integration
- Kubernetes ingress, DNS, and TLS
- Repository ownership and platform responsibility boundaries
- Cost-aware dev rebuild/destroy workflow
- Multi-environment and landing-zone-style architecture planning
