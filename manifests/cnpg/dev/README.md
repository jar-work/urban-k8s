# cnpg — dev

Postgres mínimo para dev (`postgres-dev.yaml`): 1 instancia, sin réplicas, recursos chicos. **Sin backup a propósito** — todavía no existe ni el bucket de GCS ni la Service Account para escribir ahí. Dev no lo necesita.

## Pendiente para prod (backup diario, 00:00 UTC)

Cuando se arme el ambiente de prod, agregar en `manifests/cnpg/prod/`:

1. **Bucket de GCS** para los backups (no existe todavía) — crear y decidir en qué proyecto de GCP vive.
2. **Service Account** con permiso de escritura sobre ese bucket, key JSON generada.
3. **Cargar la key en Secret Manager** (mismo patrón que `gcr-puller-credentials`), y un `ExternalSecret` que la traiga como Secret de Kubernetes en el namespace `cnpg-system` — el `Cluster` de prod referencia ese Secret por nombre en `spec.backup.barmanObjectStore.googleCredentials.applicationCredentials`, no hace falta que el Secret exista de antemano para escribir el YAML, pero sí para que el backup funcione de verdad.
4. Agregar el bloque `backup:` al `Cluster` de prod (`barmanObjectStore` apuntando al bucket + el Secret del punto 3).
5. Un `ScheduledBackup` separado, `schedule: "0 0 0 * * *"` (00:00 UTC diario, formato cron de CNPG con segundos: `segundo minuto hora día mes día-semana`).

Referencia del patrón completo (operator + cluster + backup + ExternalSecret) en `/Volumes/Projects/k8s/argo/manifests/cnpg/prod/`.
