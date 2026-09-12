# ingress-nginx

Single ingress controller for the cluster, fronted by MetalLB on the LAN. TLS terminates at the ingress using certs from `home-issuer` (internal CA) or `cloudflare-issuer` (future).

`enable-ssl-passthrough` is on (controller CLI flag via `extraArgs`) because Pinniped's supervisor + impersonation proxy use TCP passthrough on the login/kubectl hosts.

## Deploy

```sh
helmfile apply
```

## Find the ingress IP

```sh
kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

Services on the LAN are reachable at `https://<svc>.<ingress-ip>.sslip.io`.

## Cleanup

```sh
helmfile destroy
```