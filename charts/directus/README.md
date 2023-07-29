# Directus Helm Chart

Installs [Directus](https://directus.io/), the open data platform for headless content management via Helm Chart.

## Install

To install Directus, use the following commands.

1. Add Directus Helm repository:

    ```sh
    helm repo add directus https://digitalist-se.github.io/directus-helm-chart
    helm repo update
    ```

2. Install Helm Chart with `directus` release name:

    ```sh
    helm install directus directus/directus
    ```

Of course, you can use whatever release name you want.

## General production recommendations

We recommend to have three replicas at least for Directus in production use.
We also don't recommend to turn of the probes, these helps to keep Directus
active, we have had issues when it comes 'idle' and the response well be bad
on the first request, usually throwing errors.

If you have more than one replica, you should have cache in redis, so you could share
the cache.

```yaml
extraEnvVars:
  - name: CACHE_ENABLED
    value: "true"
  - name: CACHE_TTL
    value: 5m
  - name: CACHE_AUTO_PURGE
    value: "true"
  - name: CACHE_STORE
    value: redis
  - name: REDIS
    value: "redis://:mysecretpassword@directus-redis-headless:6379/1"
  ...    
```

## Healthchecks

The probes makes http requests to `/server/health/`, doing that, you may get
warnings in your pod logs for high threshold on mysql, cache or storage,
you could override the default for treshold warnings , if you can't make the
answer time improve for the services.

```yaml
extraEnvVars:
  - name: DB_HEALTHCHECK_THRESHOLD
    value: "300"
  - name: CACHE_HEALTHCHECK_THRESHOLD
    value: "300"
  - name: STORAGE_S3_HEALTHCHECK_THRESHOLD
    value: "850"
  ...    
```

## Uninstall

To uninstall and delete Directus Helm release:

```sh
helm delete directus-release 
```

## Configuration

| Parameter | Description | Default Value |
| --- | --- | --- |
| `replicaCount`   | Number of replicas  | 1  |
| `image.repository`   | Image name  | `directus/directus`   |
| `image.pullPolicy`   | Image pull policy   | `IfNotPresent`|
| `image.pullSecrets`  | Image pull secrets  | `{}`   |
| `nameOverride`   | Replaces the name of the chart | -- |
| `fullnameOverride`   | Replaces the fully qualified app name  | -- |
| `serviceAccount.create`  | Create service account | `true` |
| `serviceAccount.annotations` | ServiceAccount annotations | `{}`   |
| `serviceAccount.name`| Service account name to use, when empty will be set to created account if serviceAccount.create is set else to default| `` |
| `podAnnotations` | Pod annotations | `{}`   |
| `podSecurityContext` | Pod securityContext | `{}`   |
| `securityContext`| Deployment securityContext | `{}`   |
| `service.type`   | Kubernetes service type| `ClusterIP`   |
| `service.port`   | Kubernetes port where service is exposed  | `80`   |
| `ingress.enabled`| Enables Ingress | `false`|
| `ingress.annotations`| Ingress annotations | `{}`   |
| `ingress.hosts`  | Ingress extra paths to prepend to every host configuration| `["chart-example.local"]` |
| `ingress.tls`| Ingress TLS configuration  | `[]`   |
| `resources`  | CPU/Memory resource requests/limits| `{}`   |
| `autoscaling.enabled`| Enables Autoscaling | `false`|
| `autoscaling.minReplicas`| Set minimum number of replicas. Only applies when `autoscaling.enabled` is `true`.  | `1`|
| `autoscaling.maxReplicas`| Set maximum number of replicas Only applies when `autoscaling.enabled` is `true`.   | `100`  |
| `startupProbe.enabled`| Enables livenessProbe | `true`|
| `livenessProbe.enabled`| Enables livenessProbe | `true`|
| `readinessProbe.enabled`| Enables readinessProbe | `true`|
| `nodeSelector`   | Node labels for pod assignment | `{}`   |
| `tolerations`| List of node taints to tolerate| `[]`   |
| `affinity`| Node/Pod affinities | `{}`   |
| `extraEnvVars`   | Adds extra environment variables.  Refer to [Directus Docs](https://docs.directus.io/configuration/config-options/) for more details. | `{}`   |
| `mariadb.enabled`| Deploys MariaDB server | `true` |
| `redis.enabled`  | Deploys Redis server| `true` |
| `sidecars`| Sidecars to attach to Directus deployment| `[]`   |
| `initContainers`| initContainers to start before directus | `[]`   |
| `extraSecrets.create`| Create extra secrets| `false`   |
| `extraSecrets.data`| Secrets to add| `{}`   |
| `extraConfigMap.create`| Create extra configmap | `false`   |
| `extraConfigMap.data`| Configmap data to add | `{}`   |
| `extraVolumes`| Extra volumes to add | `[]`   |
| `extraVolumeMounts`| Extra volumemounts to add | `[]`   |

### External Database

If you want to use an external DB, you need to disable MariaDB by adding `mariadb.enabled: false` and add the necessary Environment Variables according to [Directus Documentation](https://docs.directus.io/configuration/config-options/#database).

These variables can be added by using the following values:

```yaml
extraEnvVars:
  - name: DB_CLIENT
    value: mysql
  ...

```

### External File Storage

By default, Directus stores the uploaded files on the container disk.  Thus the data will be lost when the pod is restarted.  You need to configure Directus to use an external storage adapter such as S3, Google Storage and Azure.  This can be done by adding the necessary Environment Variables as documented in [Directus Documentation](https://docs.directus.io/configuration/config-options/#file-storage).

These variables can be added by using the following values:

```yaml
extraEnvVars:
  - name: STORAGE_LOCATIONS
    value: amazon
  - name: STORAGE_AMAZON_DRIVER
    value: s3
  ...

```
