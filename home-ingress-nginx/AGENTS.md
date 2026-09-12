<!-- FOR AI AGENTS - Human readability is a side effect, not a goal -->
<!-- Managed by agent: opencode | Last updated: 2026-08-23 -->

# AGENTS.md (home-ingress-nginx)

Day-0 base component: single ingress controller, LoadBalancer fronted by MetalLB on the LAN.

## Overview
Overview of this component — see `README.md` for prose.

## Setup (deploy & teardown)

| Task | Command |
|------|---------|
| Apply | `helmfile apply` (this dir) |
| Cleanup | `helmfile destroy` |
| Find LB IP | `kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` — this becomes `<ingress-ip>` |

## Commands
Full command table lives in the Setup section; offline validation: `helm template <rel> ./charts/<name>` and `helm lint` via the alpine/helm container (see workspace root AGENTS.md).

## Key files

| File | Notes |
|------|-------|
| `charts/upstream/ingress-nginx-4.15.1.tgz` | pinned (controller v1.15.1) |
| `values/ingress-nginx-values.yaml` | LoadBalancer, `force-ssl-redirect`, **`extraArgs.enable-ssl-passthrough: "true"`** (CLI flag — required by Pinniped supervisor + impersonation proxy; the ConfigMap key is ignored by current controllers), worker-pool nodeSelector |

## Conventions & rules

- `extraArgs.enable-ssl-passthrough` must stay on (CLI flag, NOT the configmap key) — Pinniped's supervisor + kubectl proxy die without it (see `home-sso-auth-rbac` note).
- Ingress controllers other than `nginx` class are not supported; all charts use `ingressClassName: nginx`.
- The controller runs on the Lenovo worker pool (nodeSelector `node-role.kubernetes.io/worker`); the control-plane node stays kube-system only (see `talos-linux-setup/designate-node-roles.yaml`).
- Cross-repo rules: workspace root `AGENTS.md`.

## Security
- No real secrets, keys, or kubeconfigs may be committed; `*.example.yaml` + `.gitignore` are the only allowed pattern.
- `<ingress-ip>`, `<your-github-org>`, `<region>`, `<user-pool-id>` are TODOs, not values to invent.

## Checklist
- [ ] helmfile chart paths / terragrunt sources resolve (no dangling `./charts/...`)
- [ ] YAML parses (`helm template` or a yaml lint)
- [ ] 3-way sync files unchanged unless intentionally updated together
- [ ] No real secrets added (only `*.example.yaml`)
- [ ] No placeholder replaced with a fabricated value

## Examples
`base/home-ingress-nginx` is the reference helmfile pattern; the `helmfile-template` repo documents the skeleton.

## When stuck
- For pipeline/auth issues: `wiki/cicd/pipelines/troubleshoot/` and the restricted wiki runbooks.
