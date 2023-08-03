# Directus Helm Chart

This Helm chart installs [Directus](https://directus.io/), a real-time API, and App dashboard for managing SQL database content.

## Install

We strive to make the installation of Directus as easy as possible, but there are a couple of important things you should override from the defaults, namely the key and secret.

We recommend that you either override the default values file from the repository or set your keys using  `--set`.

*Disclaimer:* Please note that we are working on moving the Directus installation to be fully automated. Currently, you might encounter issues that require you to set
 `extraEnvVars` for missing values. Our plan is to have addressed all these issues in version `0.7.0`.

### Add helm repo

Add the Directus Helm repository:

```bash
helm repo add directus https://digitalist-se.github.io/directus-helm-chart
helm repo update
```

### Example 1 - Installing Directus

To install Directus, use the following command:

```bash
helm install my-release-name --set key=myrandomkey --set secret=myrandomsecret directus/directus
```

You can generate your keys as an example with the following command:

```bash
openssl rand -hex 16
```

### Example 2 - Installing Directus with overrides

```bash
helm install my-release-name -f my-overrides.yaml directus/directus
```

### Simple installation test

```bash
helm test my-release-name
```

This creates a pod that checks if the Directus service works and provides a correct answer.

## General production recommendations

Use HPA (Horizontal Pod Autoscaler) for Directus (`autoscaling: enabled: true`).

We recommend having at least three replicas for Directus in production  (`replicaCount: 3`) and setting a minimum number of replicas for HPA to 3  (`autoscaling: minReplicas: 3`) with a maximum value that matches your requirements (`autoscaling: maxReplicas: 12`)

We also advise against turning off the probes, as they help keep Directus active. We have encountered issues when it remains idle, leading to poor responses on the first request, which often results in errors.

If you have more than one replica, you should use Redis for caching, so you can share the cache.

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

The probes make HTTP requests to  `/server/health/`, which may trigger warnings in your pod logs for high thresholds on MySQL, cache, or storage. If you are unable to improve the answer time for these services, you can override the default threshold warnings.

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

To uninstall and delete the Directus Helm release:

```sh
helm delete my-release-name
```

## Configuration

| Parameter | Description | Default Value |
| --- | --- | --- |
| `replicaCount`   | Number of replicas  | 1  |
| `key`   | Unique key  | `4d596b1af3e1ae23d54110da1a377c66` (please change)  |
| `secret`   | Unique secret  | `7cf5038d95dfba34178e0c120d77d75e` (please change)  |
| `image.repository`   | Image name  | `directus/directus`   |
| `image.pullPolicy`   | Image pull policy   | `IfNotPresent`|
| `image.pullSecrets`  | Image pull secrets  | `{}`   |
| `nameOverride`   | Replaces the name of the chart | -- |
| `fullnameOverride`   | Replaces the fully qualified app name  | -- |
| `serviceAccount.create`  | Create service account | `true` |
| `serviceAccount.annotations` | ServiceAccount annotations | `{}`   |
| `serviceAccount.name`| Service account name to use, when empty will be set to the created account if serviceAccount.create is set, else to default | `` |
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
| `extraEnvVars`   | Adds extra environment variables.  Refer to [Directus Docs](https://docs.directus.io/self-hosted/config-options.html) for more details. | `{}`   |
| `mariadb.enabled`| Deploys MariaDB server | `true` |
| `redis.enabled`  | Deploys Redis server| `true` |
| `sidecars`| Sidecars to attach to Directus deployment| `[]`   |
| `initContainers`| initContainers to start before Directus | `[]`   |
| `extraSecrets.create`| Create extra secrets| `false`   |
| `extraSecrets.data`| Secrets to add| `{}`   |
| `extraConfigMap.create`| Create extra config map | `false`   |
| `extraConfigMap.data`| Config map data to add | `{}`   |
| `extraVolumes`| Extra volumes to add | `[]`   |
| `extraVolumeMounts`| Extra volume mounts to add | `[]`   |
| `snapshot`| Take schema snapshot at init  | `false`   |

### External Database

If you want to use an external database, you need to disable MariaDB by adding `mariadb.enabled: false` and set the necessary environment variables according to the [Directus Documentation](https://docs.directus.io/self-hosted/config-options.html#database).

These variables can be added using the following values:

```yaml
extraEnvVars:
  - name: DB_CLIENT
    value: mysql
  ...

```

### External File Storage

By default, Directus stores uploaded files on the container disk. As a result, the data will be lost when the pod is restarted. To avoid this, you need to configure Directus to use an external storage adapter, such as S3, Google Storage, or Azure. This can be achieved by adding the necessary environment variables as documented in the [Directus Documentation](https://docs.directus.io/self-hosted/config-options.html#file-storage).

You can add these variables using the following values:

```yaml
extraEnvVars:
  - name: STORAGE_LOCATIONS
    value: amazon
  - name: STORAGE_AMAZON_DRIVER
    value: s3
  ...

```

## Overrides for docker image

We are overriding the Docker defaults by moving `directus bootstrap` o an initContainer and only running `directus start` in the main container.

## Plan

We are gradually moving more configuration settings to `values.yaml`  instead of requiring all `extraEnvVars` to be set during the initial installation of Directus. This work is in progress, and major changes will be reflected in the version number of the chart.

