# Platform status — October 2026

JSAPPINF now has a shared DEV/STAGE/PROD platform structure, environment-specific state configuration, shared Helm charts and environment-specific GitOps desired state.

## October 2 operational evidence

The shared Terraform root provisioned the DEV foundation and cluster runtime. Three EKS 1.35 nodes became ready. The database provisioner completed twice; the four current database secret versions remained unchanged after the second invocation.

Argo CD deployed monitoring and the four application services with the intended image tags. The UI returned HTTP 200 over HTTPS, Grafana redirected to login and HTTP redirected to HTTPS. Explicit ALB proxy trust restored issuance of the secure UI session cookie. Approved customer identity configuration reached user-service.

The teardown removed Gateway/ALB resources while their controller was available, cascaded GitOps applications, then removed Terraform runtime, services, EKS, networking and selected disposable resources. Protected KMS keys, backend state, Cognito identity and the protected Grafana secret were retained.

These observations establish platform lifecycle and deployment evidence. The complete customer ordering journey remains application development work.

## October 5 operating model

Manual runbooks cover per-environment backend bootstrap, identity preparation, foundation, provisioning, runtime and GitOps. Guarded shutdown scripts encode dependency order, environment/account/backend binding and protected-resource checks. STAGE/PROD retirement requires an explicit approval record and database recovery safeguards.

Related Terraform declarations were consolidated within the same root, preserving resource addresses. The old `infra-next` directory was archived outside the implementation repository with ignored private files retained; the active backend key and independent DEV identity directory were preserved.

## Further work

Local application development, customer-flow fixes, security-group review, edge testing and recovery exercises are the next focus. CloudFront/WAF and Karpenter remain optional capabilities; earlier experiments are documented separately.
