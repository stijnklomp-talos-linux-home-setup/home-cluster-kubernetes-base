# MetalLB

Bare-metal LoadBalancer for the home cluster (L2 mode). Required before ingress-nginx can get an IP.

## Deploy

```sh
helmfile apply
```

Gives `LoadBalancer` services an address from `192.168.1.200-192.168.1.230` (see `kustomizations/metallb-config/ipaddresspool.yaml`; adjust to the LAN subnet / DHCP free range).

## Cleanup

```sh
helmfile destroy
```