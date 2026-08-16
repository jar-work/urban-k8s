# cloudflared — dev

Túnel de Cloudflare corriendo dentro del cluster, apuntado a Traefik. Gestionado por ArgoCD vía `apps/cloudflared.yaml` (Application de path simple, sin Helm — son manifiestos planos).

## Orden de instalación

Depende de `external-secrets` y su `ClusterSecretStore` (`gcp-secret-manager`) ya instalados y sincronizados — el `external-secret.yaml` de acá los referencia para traer el token. Sin eso, el `ExternalSecret` va a quedar en estado de error hasta que resuelva.

1. `external-secrets` (`apps/external-secrets.yaml` + `apps/external-secrets-config.yaml`) — ver `manifests/external-secrets/dev/README.md`.
2. Cargar el token del túnel en GCP Secret Manager, bajo la key `cloudflared-tunnel-token-dev` (el nombre que espera `external-secret.yaml`). El token se obtiene con:
   ```bash
   cloudflared tunnel token <TUNNEL-ID>
   ```
3. Registrar esta Application en ArgoCD:
   ```bash
   sudo k3s kubectl apply -f ../../../apps/cloudflared.yaml
   ```

Ya no hace falta crear el Secret a mano con `kubectl create secret` — lo genera el `ExternalSecret` a partir de GCP Secret Manager.

## Verificar

```bash
sudo k3s kubectl -n cloudflare-tunnel get externalsecret
sudo k3s kubectl -n cloudflare-tunnel get pods
sudo k3s kubectl -n cloudflare-tunnel logs -l app=cloudflared
```

Buscar `Registered tunnel connection` en los logs para confirmar que conectó.
