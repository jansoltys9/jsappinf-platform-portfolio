# Karpenter Capacity Flow

This document describes the optional Karpenter capacity model used in the JSAPPINF platform.

Karpenter is treated as a validation and scaling component, not as something that must always run in the cost-aware dev profile.

---

## Purpose

The goal of the Karpenter design is to demonstrate production-like capacity planning while keeping the development environment affordable.

The model separates:

```text
stable baseline workloads
burst workloads
ON_DEMAND capacity
SPOT capacity
```

---

## Stable and Burst Capacity Model

The intended application capacity model is:

```text
stable replicas
  -> ON_DEMAND capacity

burst replicas
  -> SPOT capacity
```

Example:

```text
product-service-stable
  -> apps-ondemand NodePool

order-service-stable
  -> apps-ondemand NodePool

product-service-burst
  -> apps-spot NodePool

order-service-burst
  -> apps-spot NodePool
```

---

## NodePool Intent

### apps-ondemand

Purpose:

```text
stable application replicas
baseline capacity
more predictable availability
ON_DEMAND EC2 instances
```

### apps-spot

Purpose:

```text
burst application replicas
cost-optimized capacity
SPOT EC2 instances
future HPA target
```

---

## Dev Default

The default dev profile should not run unnecessary SPOT burst capacity.

Recommended dev default:

```text
Karpenter disabled or idle
apps-spot NodePool has no active nodes
burst replicas disabled
lower node count preferred
```

Example Helm replica strategy:

```yaml
replicaStrategy:
  totalReplicas: 2
  stableOnDemandReplicas: 2
```

Meaning:

```text
stable replicas = 2
burst replicas  = 0
```

---

## Validation Mode

Karpenter validation can temporarily enable burst replicas.

Example validation strategy:

```yaml
replicaStrategy:
  totalReplicas: 4
  stableOnDemandReplicas: 2
```

Meaning:

```text
stable replicas = 2
burst replicas  = 2
```

This validates:

```text
ON_DEMAND application placement
SPOT application placement
taints and tolerations
node selectors
NodeClaims
scale-out behavior
scale-down behavior
```

---

## Cost-Aware Decision

The project avoids running unnecessary EC2 nodes in normal dev mode.

Validated capability:

```text
Karpenter ON_DEMAND/SPOT scheduling model
stable/burst workload placement
optional autoscaling direction
```

Default behavior:

```text
Karpenter disabled or idle
SPOT burst disabled
daily destroy/rebuild workflow
```

---

## Production-Like Direction

In a production-like profile, Karpenter can be used more intentionally.

Possible future direction:

```text
baseline workloads on ON_DEMAND capacity
burst workloads on SPOT capacity
HPA scales burst deployments
topology spread constraints
anti-affinity for availability
explicit SPOT interruption testing
environment-specific capacity profiles
```

---

## Portfolio Summary

The Karpenter capacity model demonstrates that JSAPPINF is not only about running workloads, but also about thinking through workload placement, cost control, scaling behavior, and production-like capacity strategy.

The key idea is:

```text
keep dev cheap by default
validate advanced capacity patterns explicitly
do not run unnecessary nodes just to show architecture
```
