<!-- FOR AI AGENTS - Human readability is a side effect, not a goal -->
<!-- Managed by agent: opencode | Last updated: 2026-09-12 -->

# AGENTS.md (home-cluster-kubernetes-base)

Day-0 base layer of the home cluster (helmfile era — everything after this goes through ArgoCD). Monorepo bundling the four former `base/home-*` repos; each subdirectory is its own helmfile project with its own AGENTS.md.

## Overview
Overview of this repo — see `README.md` for prose.

## Setup (deploy & teardown)

Deploy in dependency order, `helmfile apply` inside each component dir:

| Task | Command |
|------|---------|
| MetalLB | `cd home-metallb && helmfile apply` (must be first — nothing else gets an IP without it) |
| cert-manager | `cd home-cert-manager && helmfile apply` |
| Longhorn | `cd home-longhorn && helmfile apply` |
| ingress-nginx | `cd home-ingress-nginx && helmfile apply` (needs MetalLB) |
| Cleanup | `helmfile destroy` per dir, reverse order |

## Commands
Full command table lives in the Setup section; offline validation: `helm template <rel> ./charts/<name>` and `helm lint` via the alpine/helm container (see workspace root AGENTS.md).

## Key files

| Path | Notes |
|------|-------|
| `home-metallb/` | L2 LoadBalancer, pool `192.168.1.200-192.168.1.230` (adjust to LAN) |
| `home-cert-manager/` | cert-manager + ClusterIssuers; `home-issuer` internal CA for every LAN ingress |
| `home-longhorn/` | default StorageClass `longhorn`; needed by Tekton workspaces, plans PVC, stateful services |
| `home-ingress-nginx/` | single ingress controller; `enable-ssl-passthrough` on (Pinniped requirement) |

## Conventions & rules

- No root `helmfile.yaml` — each component dir is a standalone helmfile project (it is also its own git-tracked unit).
- Deploy order matters: `home-metallb` first, `home-ingress-nginx` last (it takes a MetalLB address).
- Component-specific rules live in each subdir's AGENTS.md — read it before editing that component.
- Never edit vendored `charts/upstream/*.tgz` in place; refresh via documented `helm pull`.
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
`home-ingress-nginx` is the reference helmfile pattern; the `helmfile-template` repo documents the skeleton.

## When stuck
- For pipeline/auth issues: `wiki/cicd/pipelines/troubleshoot/` and the restricted wiki runbooks.