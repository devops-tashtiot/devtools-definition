# devtools-definition

Environment-specific Helm values overrides for the devtools platform (Jira, Bitbucket,
Confluence, ArgoCD, Artifactory, Xray) deployed from
[`devtools-provision`](https://github.com/devops-tashtiot/devtools-provision) — the
configuration half of that provision/definition pair. The cluster-infra equivalent is
[`clusters-provision`](https://github.com/devops-tashtiot/clusters-provision)/
[`clusters-definition`](https://github.com/devops-tashtiot/clusters-definition).

## Why a separate repo instead of one

Keeping "what to deploy" (the chart, in `devtools-provision`) separate from "how to
configure it per environment" (this repo) means the same chart runs unmodified against a
local Kind cluster and against EKS — only the values differ (service type/annotations,
resource limits, ingress hosts, credentials). It also means environment secrets and
per-cloud choices never require a chart change, or a re-review of the chart itself, to
update.

## How it works

ArgoCD's `ApplicationSet` (`applicationset.yaml`) auto-discovers every tool directory under
`devtools-provision/devtools/*` and merges two Helm value sources per tool:

1. `devtools-provision/devtools/<tool>/values.yaml` — defaults, same across every
   environment
2. `devtools/<tool>/values.yaml` (this repo) — environment-specific overrides (EKS-specific
   here), applied on top

Only override what differs per environment here — don't duplicate base values.

## Currently tracked tools

See [`CLAUDE.md`](./CLAUDE.md) for the current table and common override patterns
(LoadBalancer/NLB service type, resource bumps), and the `add-devtool` skill for onboarding
a new tool.
