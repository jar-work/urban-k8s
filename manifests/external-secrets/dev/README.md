# external-secrets — dev

Sigue el mismo patrón que `apps/cloudflared`: el `Application` de ArgoCD gestiona el operador (Helm chart, `apps/external-secrets.yaml`) y el `ClusterSecretStore`/`ExternalSecret`s de este directorio (`apps/external-secrets-config.yaml`), pero el secreto que le da acceso al operador a GCP **no está versionado acá** — se crea a mano, una sola vez por cluster.

## 1. Proyecto de GCP

`cluster-secret-store.yaml` usa el proyecto `urbanqualityproject-dev`.

## 2. Crear la Service Account Key en GCP

Una Service Account con rol `Secret Manager Secret Accessor` sobre `urbanqualityproject-dev`, y una key JSON descargada.

## 3. Cargar la key como Secret en el cluster

```bash
sudo k3s kubectl create namespace external-secrets
sudo k3s kubectl create secret generic gcpsm-secret \
  --namespace external-secrets \
  --from-file=secret-access-credentials=<path-a-la-key.json>
```

## 4. Registrar las dos Applications en ArgoCD

```bash
sudo k3s kubectl apply -f ../../../apps/external-secrets.yaml
sudo k3s kubectl apply -f ../../../apps/external-secrets-config.yaml
```

Sin `syncPolicy.automated` — quedan en sync manual, hay que aprobar el sync desde la UI o CLI de ArgoCD la primera vez (y cada vez que cambie algo), igual que en el repo de referencia.
