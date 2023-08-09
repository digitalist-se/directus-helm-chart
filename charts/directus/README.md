# Directus Helm Chart by Digitalist

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/directus)](https://artifacthub.io/packages/search?repo=directus)
[![checkov](https://github.com/digitalist-se/directus-helm-chart/actions/workflows/checkov.yaml/badge.svg)](https://github.com/digitalist-se/directus-helm-chart/actions/workflows/checkov.yaml?branch=main)
[![Lint&Test chart](https://github.com/digitalist-se/directus-helm-chart/actions/workflows/lint-test.yaml/badge.svg)](https://github.com/digitalist-se/directus-helm-chart/actions/workflows/lint-test.yaml?branch=main)
[![Unit test chart](https://github.com/digitalist-se/directus-helm-chart/actions/workflows/unit-test.yaml/badge.svg)](https://github.com/digitalist-se/directus-helm-chart/actions/workflows/unit-test.yaml?branch=main)

This Helm chart installs [Directus](https://directus.io/), a real-time API, and App dashboard for managing SQL database content.

## Un-official

Please note, this is an un-official Helm chart from Digitalist.

## License

The Directus software is licensed with a Business Source License 1.1, © 2023 Monospace, Inc. This chart, however, is licensed with GPL v.3 license. This means that for using and distributing Directus itself you would refer to the license of Directus, not the Helm chart license.

## Install

We strive to make the installation of Directus as easy as possible, but there are a couple of important things you should override from the defaults, namely the key and secret.

We recommend that you either override the default values file from the repository or set your keys using  `--set`.

### File storage

By default, no persistance storage is set, in general, you are recommended to use [External File Storage](README.md#external-file-storage).

If you want to use internal persistance file storage (disk), you need to enable it:

```yaml
persistence:
  enabled: true
```

If you desire to have more than one replica of Directus, it is essential to use a storage class that facilitates ReadWriteMany functionality. Without this, the deployment scalability would be restricted, as additional replicas
would never start:

```yaml
persistence:
  ...
  storageClass: "longhorn" #or any other class supporting ReadWriteMany
  accessModes: ReadWriteMany
  ...
```

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

If you have more than one replica, you should use Redis for caching, so you can share the cache. This is enabled by default.

```yaml
cache:
  enabled: true
  ttl: "5m"
  autoPurge: true
  store: "redis"

```

## Health checks

The probes make HTTP requests to  `/server/health/`, which may trigger warnings in your pod logs for high thresholds on MySQL, cache, or storage. If you are unable to improve the response time for these services, you can override the default threshold warnings.

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
| `admin.email`   | Email for admin user  | -- |
| `admin.password`   | Password for admin user  | -- |
| `public.url`   | Public URL for Directus  | -- |
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
| `persistence.enabled` | If to enable  persistence storage | `false`   |
| `persistence.existingClaim` | Existing Persistent Volume Claim to use |  ``  |
| `persistence.storageClass` | storageClass for volume  |  `-`  |
| `persistence.annotations` | Annotations to add to volume  |  `{}`  |
| `persistence.accessModes` | accessMode for Persistent Volume   |  `ReadWriteOnce`  |
| `persistence.size`   | Size for Volume   |  `8Gi`  |
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
| `generateEnvVars.mariadb`| Creates env. variables from mariadb values | `true`   |
| `generateEnvVars.redis`| Creates env. variables from redis values | `true`   |
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
| `log.level`| Logging level | `info`   |
| `log.style`| Logging style | `pretty`   |
| `cache.enabled`| To enable caching | `true`   |
| `cache.ttl`| Time to live for cache objects | `5m`   |
| `cache.autoPurge`| Auto purge cache | `true`   |
| `cache.store`| Where to store cache | `redis`   |

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

By default, with the helm charts defaults, Directus stores uploaded files on the pod storage in an empty dir. As a result, the data will be lost when the pod is restarted. To avoid this, you need to configure Directus to use an external storage adapter, such as S3, Google Storage, or Azure, or enable persistent storage (only recommended if you have storage class with `ReadWriteMany` capacity).
For external file storage, you need to add the necessary environment variables as documented in the [Directus Documentation](https://docs.directus.io/self-hosted/config-options.html#file-storage).

You can add these variables like this:

```yaml
extraEnvVars:
  - name: STORAGE_LOCATIONS
    value: amazon
  - name: STORAGE_AMAZON_DRIVER
    value: s3
  ...

```

## Overrides for docker image

We are overriding the Docker defaults by moving `directus bootstrap` to an initContainer and only running `directus start` in the main container.

## Helm unit testing

Tests are in `charts/directus/tests`

First install the helm plugin if you don't have it.

```bash
helm plugin install https://github.com/helm-unittest/helm-unittest.git
```

In the charts dir (`charts/directus`), run:

````bash
helm unittest . --strict
```

## Plan

We are gradually moving more configuration settings to `values.yaml` so we don't depend to much on `extraEnvVars`.
We also plan to support schema import on install and update.
