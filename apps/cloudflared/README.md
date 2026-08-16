# cloudflared

Tunnel de Cloudflare corriendo dentro del cluster, apuntado a Traefik. Aplicado manualmente por ahora (no gestionado por ArgoCD todavía).

El `Secret` con el token del túnel **no está versionado acá** — se crea a mano, una vez por cluster:

```bash
sudo k3s kubectl create secret generic cloudflared-token \
  --namespace cloudflare-tunnel \
  --from-literal=TUNNEL_TOKEN='<token de "cloudflared tunnel token <TUNNEL-ID>">'
```

(Si el namespace `cloudflare-tunnel` todavía no existe, aplicar primero `namespace.yaml`, o dejar que el `create secret` de arriba lo cree con `--namespace` solo falla si no existe — aplicar `namespace.yaml` antes.)

Aplicar todo:

```bash
sudo k3s kubectl apply -f namespace.yaml
# crear el Secret (ver arriba)
sudo k3s kubectl apply -f configmap.yaml
sudo k3s kubectl apply -f deployment.yaml
```

Verificar:

```bash
sudo k3s kubectl -n cloudflare-tunnel get pods
sudo k3s kubectl -n cloudflare-tunnel logs -l app=cloudflared
```

Buscar en los logs una línea tipo `Registered tunnel connection` para confirmar que conectó.
