# Edge Security – CloudFront and AWS WAF

This document describes the implemented edge security source and remaining routing proposals for the JSAPPINF platform.

CloudFront and AWS WAF integration with an ALB origin was previously validated. NLB and ingress-nginx belong to an earlier platform iteration.

As of 2026-09-17, DEV runtime is intentionally OFF for cost control. Runtime descriptions below refer to source configuration or previously validated capabilities, not currently running workloads.

---

## Current Exposure Model

Current source configures `app.dev.jsapp365.com` for the UI and `api.dev.jsapp365.com/users`, `/products` and `/orders` through ALB and Gateway API. These are configured routes, not a claim of current live availability.

Previous lab/debug routing model:

```text
ui.jsapp365.com
user.jsapp365.com
product.jsapp365.com
order.jsapp365.com
```

This model is useful for learning, debugging and validating each service independently.

However, it is not the preferred long-term production-like public routing model.

---

## Target Public Routing Model

The target model should expose a clean product entrypoint and hide individual backend microservices behind path-based routing.

Target production-like model:

```text
jsapp365.com
  -> frontend / UI

jsapp365.com/api/users
  -> user-service

jsapp365.com/api/products
  -> product-service

jsapp365.com/api/orders
  -> order-service
```

Alternative future single-host dev proposal (the current source uses app.dev and api.dev as described above):

```text
dev.jsapp365.com
  -> frontend / UI

dev.jsapp365.com/api/users
  -> user-service

dev.jsapp365.com/api/products
  -> product-service

dev.jsapp365.com/api/orders
  -> order-service
```

Microservices remain internal implementation details.

Internal Kubernetes service discovery can still use names such as:

```text
user-service.user-service.svc.cluster.local
product-service.product-service.svc.cluster.local
order-service.order-service.svc.cluster.local
ui-service.ui-service.svc.cluster.local
```

---

## Why CloudFront + AWS WAF

The current edge source uses an ALB origin. AWS WAF is associated with CloudFront.

Target edge architecture:

```text
Internet
  -> CloudFront
  -> AWS WAF Web ACL
  -> Application Load Balancer (configured through Gateway API / HTTPRoute)
  -> JSAPP services
```

This reflects the ALB/Gateway API integration already implemented in source.

---

## Initial WAF Baseline

The first WAF baseline should be intentionally simple and cost-aware.

Initial managed rules:

```text
AWSManagedRulesCommonRuleSet
AWSManagedRulesKnownBadInputsRuleSet
basic rate-based rule
```

Initially disabled:

```text
Bot Control
CAPTCHA
Challenge
full WAF request logging
aggressive geo blocking
large custom rule sets
```

This keeps the dev platform affordable while still demonstrating production-like security thinking.

---

## CloudFront Behavior

Initial CloudFront behavior should focus on safe dynamic application traffic.

Recommended first behavior:

```text
viewer protocol policy: redirect-to-https
cache policy: caching disabled
origin request policy: forward viewer context
allowed methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
```

Static asset caching can be added later for paths such as:

```text
/assets/*
```

---

## DNS Cutover Strategy

The edge layer should be introduced carefully.

Recommended phases:

```text
1. Create CloudFront and AWS WAF without changing app DNS.
2. Validate CloudFront default domain.
3. Validate Host header routing.
4. Validate UI and API paths.
5. Cut over DNS only after validation.
```

This describes the staged cutover approach; the July 2026 cutover was previously validated.

---

## Current Implementation Status

Current project status:

```text
CloudFront + WAF infrastructure implemented
edge-waf component registered
ALB-origin integration previously validated
DNS cutover workflow previously validated
DEV runtime currently OFF
```

Historical infrastructure and cutover validation does not imply current live operation. Edge exposure is not kept continuously active in the cost-aware dev environment.

---

## Cost-Aware Decision

The dev environment is frequently destroyed or stopped when not used.

For this reason, CloudFront/WAF should not be enabled by default in dev.

Recommended dev default:

```text
enable_edge_waf = false
```

Enable only when testing edge security:

```text
enable_edge_waf = true
```

This keeps the environment cost-aware while still allowing realistic edge security validation.

---

## Future Improvements

Planned improvements:

```text
single public product entrypoint
/api path-based backend routing
CloudFront in front of clean public entrypoints
WAF metrics review
optional WAF logging
admin endpoint separation
stronger origin TLS model
```

---

## Portfolio Summary

The CloudFront + AWS WAF design shows a production-like security direction:

```text
hide microservices behind clean public routing
protect public entrypoints at the edge
keep Kubernetes ingress flexible
introduce WAF without breaking the existing platform
control cost by enabling edge security only when needed in dev
```
