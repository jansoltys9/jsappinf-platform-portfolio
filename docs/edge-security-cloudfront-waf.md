# Edge Security – CloudFront and AWS WAF

This document describes the planned edge security direction for the JSAPPINF platform.

The goal is to add a production-like security layer in front of the Kubernetes ingress stack without breaking the existing working NLB and ingress-nginx model.

---

## Current Exposure Model

The current lab environment exposes applications through ingress-nginx behind an AWS Network Load Balancer.

Current lab/debug routing model:

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

Target dev model:

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

The current ingress-nginx controller is exposed through a Network Load Balancer.

AWS WAF is not attached directly to the NLB in this design. Instead, AWS WAF is attached to CloudFront.

Target edge architecture:

```text
Internet
  -> CloudFront
  -> AWS WAF Web ACL
  -> existing NLB
  -> ingress-nginx
  -> Kubernetes Ingress
  -> JSAPP services
```

This allows the platform to keep the current Kubernetes ingress architecture while adding an edge security layer.

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

The first implementation should avoid immediate public DNS cutover.

---

## Current Implementation Status

Current project status:

```text
CloudFront + WAF design documented
edge-waf component registered
Terraform skeleton created
root wiring added in disabled mode
Terraform plan validated with edge_waf enabled
apply/testing intentionally deferred
```

This means the design and Terraform structure are ready, but the actual CloudFront distribution and WAF are not required to run continuously in the cost-aware dev environment.

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
possible migration from NLB/ingress-nginx to ALB-based model if needed
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
