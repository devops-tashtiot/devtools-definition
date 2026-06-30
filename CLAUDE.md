# CLAUDE.md — devtools-definition

This repo holds environment-specific Helm values overrides for the devtools platform. It is the configuration layer in a three-repo GitOps architecture.

## Role in the Architecture

ArgoCD's ApplicationSet (defined in `devtools-labs`) deploys each tool using **two sources**:

1. The Helm chart from `devtools-provision` (base chart + default `values.yaml`)
2. A reference named `$definition` pointing to this repo

Values are merged: `devtools-provision/devtools/<tool>/values.yaml` is applied first, then `devtools-definition/devtools/<tool>/values.yaml` overrides on top.

This means: **don't duplicate base values here — only override what differs per environment.**

## Repository Structure

```
.
├── application.yaml       # ArgoCD Application — bootstraps the ApplicationSet from GitHub
├── applicationset.yaml    # ApplicationSet — auto-discovers devtools/* via in-cluster Gitea
└── devtools/
    └── <tool>/
        └── values.yaml   # EKS-specific overrides for the tool's Helm chart
```

Currently tracked tools:

| Directory | Overrides |
|---|---|
| `devtools/argocd/` | LoadBalancer service type, NLB annotations, resource limits for EKS |

## Adding Overrides for a New Tool

1. Create `devtools/<toolname>/values.yaml`
2. Include only values that differ from the defaults in `devtools-provision/devtools/<toolname>/values.yaml`
3. Commit and push to `main` — ArgoCD will pick up the change automatically on the next sync

## Common Override Patterns

**Changing service type for EKS (NLB exposure):**
```yaml
<chart-name>:
  server:
    service:
      type: LoadBalancer
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
        service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
```

**Bumping resource limits:**
```yaml
<chart-name>:
  controller:
    resources:
      requests:
        cpu: 200m
        memory: 512Mi
      limits:
        cpu: 500m
        memory: 1Gi
```

## What Belongs Here vs. devtools-provision

| Concern | Where it lives |
|---|---|
| Chart dependency, chart version | `devtools-provision/devtools/<tool>/Chart.yaml` |
| Default values (local Kind cluster) | `devtools-provision/devtools/<tool>/values.yaml` |
| EKS / cloud-specific overrides | `devtools-definition/devtools/<tool>/values.yaml` (this repo) |
| Cluster infrastructure | `devtools-labs` |
