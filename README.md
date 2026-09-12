# Kubernetes base layer

Day-0 components of the home cluster, deployed with helmfile before ArgoCD takes over. One component per directory; each is a self-contained helmfile project.

## Components

| Directory | What it does |
|-----------|--------------|
| `home-metallb` | Bare-metal LoadBalancer (L2) — gives services IPs from `192.168.1.200-192.168.1.230` |
| `home-cert-manager` | TLS certs; internal CA `home-issuer` for all LAN hosts, Cloudflare DNS-01 reserved for later |
| `home-longhorn` | Distributed storage, default StorageClass `longhorn` (Tekton workspaces, plans PVC, stateful services) |
| `home-ingress-nginx` | Single ingress controller; `ssl-passthrough` on for Pinniped |

## Deploy

In this order (ingress-nginx needs MetalLB; the rest are independent):

```sh
cd home-metallb && helmfile apply
cd home-cert-manager && helmfile apply
cd home-longhorn && helmfile apply
cd home-ingress-nginx && helmfile apply
```

Services are reachable at `https://<svc>.<ingress-ip>.sslip.io` once ingress-nginx has its MetalLB address.

## Cleanup

```sh
helmfile destroy
```

per directory, in reverse order.