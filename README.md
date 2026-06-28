# tuwunel Helm Chart

[![Helm Chart](https://img.shields.io/badge/helm-tuwunel-blue)](https://github.com/alexohneander/tuwunel-helm)
[![App Version](https://img.shields.io/badge/app%20version-v1.8.0-green)](https://github.com/matrix-construct/tuwunel)

A Helm chart for [tuwunel][homepage] — a featureful [Matrix][matrix] homeserver, forked from [conduwuit][conduwuit].

## Prerequisites

- Kubernetes 1.21+
- Helm 3.x
- A working storage class for persistent volumes (if not using an existing PVC)

## Installing the Chart

Install the chart with the release name `my-release`:

```console
helm install my-release \
  --set config.server_name=matrix.example.org \
  oci://ghcr.io/alexohneander/tuwunel-helm/tuwunel
```

> **Note:** Set `config.server_name` to your public Matrix domain before installing. This value cannot be changed after the first run without data migration.

## Uninstalling the Chart

```console
helm delete my-release
```

This removes all Kubernetes components associated with the chart and deletes the release. Persistent volumes are **not** deleted automatically.

## Configuration

Parameters can be set via `--set key=value` or by providing a custom `values.yaml`:

```console
helm install my-release -f my-values.yaml \
  oci://ghcr.io/alexohneander/tuwunel-helm/tuwunel
```

Refer to the [values.yaml](values.yaml) file for all available options and their defaults.

### TOML Config

To enable the optional TOML configuration file, set `tomlConfig.enabled=true`. The chart creates a ConfigMap and mounts it into the pod, then points `TUWUNEL_CONFIG` to the mounted file path.

When TOML config is enabled, all other `config.*` values are ignored and the application will use the TOML file for configuration.

Example:

```yaml
tomlConfig:
  enabled: true
  mountPath: /etc/tuwunel.toml
  configName: tuwunel.toml
  data: |
    # Your custom TUWUNEL config
    [server]
    server_name = "matrix.example.org"
```

### Parameters

| Parameter                          | Description                                                                                 | Default                            |
| ---------------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------- |
| `image.repository`                 | Image repository                                                                            | `ghcr.io/matrix-construct/tuwunel` |
| `image.tag`                        | Image tag. Available tags listed [here][docker].                                            | `v1.8.0`                           |
| `image.pullPolicy`                 | Image pull policy                                                                           | `IfNotPresent`                     |
| `config.server_name`               | Public Matrix server name (e.g. `matrix.example.org`)                                      | `your.server.name`                 |
| `config.max_request_size`          | Maximum upload size in bytes                                                                | `20000000` (20 MB)                 |
| `config.allow_registration`        | Allow users to self-register new accounts                                                   | `false`                            |
| `config.registration_token`        | Registration token required when registration is enabled                                   | `supa-dupa-secret-token`           |
| `config.allow_federation`          | Allow federation with other Matrix servers                                                  | `false`                            |
| `config.trusted_servers`           | Servers to trust when federating; `matrix.org` is a common choice                          | `[]`                               |
| `extraLabels`                      | Additional labels applied to all created resources                                          | `{}`                               |
| `service.annotations`              | Annotations for the Service resource                                                        | `{}`                               |
| `service.type`                     | Service type                                                                                | `ClusterIP`                        |
| `service.clusterIP`                | Cluster IP; if blank, assigned randomly from the cluster CIDR range                        | `None`                             |
| `service.port`                     | Port exposed by the service                                                                 | `80`                               |
| `service.externalIPs`              | External IPs for the service                                                                | `[]`                               |
| `service.loadBalancerIP`           | Load balancer IP                                                                            | `""`                               |
| `service.loadBalancerSourceRanges` | IP CIDRs allowed to access the load balancer (if supported)                                | `[]`                               |
| `ingress.enabled`                  | Deploy an Ingress resource                                                                  | `false`                            |
| `ingress.class`                    | Ingress class (included in annotations)                                                     | `""`                               |
| `ingress.annotations`              | Ingress annotations                                                                         | `{}`                               |
| `ingress.path`                     | Ingress path                                                                                | `/`                                |
| `ingress.hosts`                    | Ingress accepted hostnames                                                                  | `[tuwunel]`                        |
| `ingress.tls`                      | Configure TLS for the Ingress                                                               | `false`                            |
| `gateway.enabled`                  | Deploy Gateway API resources (HTTPRoute)                                                    | `false`                            |
| `gateway.create`                   | Also create a Gateway resource (in addition to the HTTPRoute)                               | `false`                            |
| `gateway.name`                     | Existing Gateway name, or the name to use when `gateway.create=true`                        | `""`                               |
| `gateway.namespace`                | Namespace of the referenced or created Gateway                                              | release namespace                  |
| `gateway.className`                | GatewayClass name for created Gateway resources                                             | `""`                               |
| `gateway.sectionName`              | Optional listener name to target on the parent Gateway                                      | `""`                               |
| `gateway.hostnames`                | Hostnames accepted by the HTTPRoute                                                         | `[tuwunel]`                        |
| `gateway.path`                     | HTTPRoute path match                                                                        | `/`                                |
| `gateway.pathType`                 | HTTPRoute path match type                                                                   | `PathPrefix`                       |
| `persistence.data.enabled`         | Use a persistent volume to store data                                                       | `true`                             |
| `persistence.data.size`            | Size of the persistent volume claim                                                         | `16Gi`                             |
| `persistence.data.existingClaim`   | Use an existing PVC to persist data                                                         | `""`                               |
| `persistence.data.storageClass`    | Storage class for the persistent volume claim                                               | `""`                               |
| `persistence.data.accessMode`      | PVC access mode                                                                             | `ReadWriteOnce`                    |
| `resources.requests`               | CPU/Memory resource requests                                                                | `1 CPU / 256 MiB`                  |
| `resources.limits`                 | CPU/Memory resource limits                                                                  | `2 CPU / 512 MiB`                  |
| `nodeSelector`                     | Node labels for pod assignment                                                              | `{}`                               |
| `tolerations`                      | Tolerations for pod assignment                                                              | `[]`                               |
| `affinity`                         | Affinity settings for pod assignment                                                        | `{}`                               |
| `tomlConfig.enabled`               | Enable optional TOML configuration file                                                     | `false`                            |
| `tomlConfig.mountPath`             | Target path inside the container for the mounted TOML file                                  | `/etc/tuwunel.toml`                |
| `tomlConfig.configName`            | ConfigMap key and filename used for the mounted TOML file                                  | `tuwunel.toml`                     |
| `tomlConfig.data`                  | TOML file contents stored in the generated ConfigMap                                       | `""`                               |

## Gateway API

Tuwunel supports the [Kubernetes Gateway API][gateway-api] as an alternative to Ingress.

### Use an existing Gateway

If a shared Gateway already exists in your cluster, enable only the HTTPRoute:

```yaml
gateway:
  enabled: true
  name: shared-gateway
  namespace: infra
  sectionName: http
  hostnames:
    - matrix.example.org
```

### Create a new Gateway

To have this chart also create the Gateway resource, enable `gateway.create` and specify the GatewayClass:

```yaml
gateway:
  enabled: true
  create: true
  className: cilium
  hostnames:
    - matrix.example.org
```

[docker]: https://github.com/matrix-construct/tuwunel/pkgs/container/tuwunel
[github]: https://github.com/matrix-construct/tuwunel
[homepage]: https://tuwunel.chat/
[matrix]: https://matrix.org/
[conduwuit]: https://gitlab.cronce.io/charts/conduwuit
[gateway-api]: https://gateway-api.sigs.k8s.io/
