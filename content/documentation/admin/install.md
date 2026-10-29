---
title: Installation
description: Install Deckhouse Development Portal with internal, external, or managed PostgreSQL, Redis, and ClickHouse instances.
weight: 11
---

Deckhouse Development Portal (DDP) is installed by the `development-platform` module in a Deckhouse Platform cluster.
Before installing it, check the [resource requirements](resource-requirements/).

Managed instances are recommended for [demo installations](#installation-with-managed-instances); [external instances](#installation-with-external-instances) are recommended for production.
[Internal instances](#installation-with-internal-instances) are available as a separate installation option.

Set `managed`, `external` or `internal` separately for PostgreSQL, Redis and ClickHouse; modes can be combined.

The `postgres.mode`, `redis.mode` and `clickhouse.mode` parameters are required and have no default value.
Without them, installation and upgrade fail settings validation.

ClickHouse stores the historical data of the portal.
To run without features that require ClickHouse, explicitly set `clickhouse.mode: external` and leave `clickhouse.host` empty, as described [below](#running-without-clickhouse).

## Installation with managed instances

In the `managed` mode, the module creates PostgreSQL, Valkey (Redis-compatible) and ClickHouse instances.
Before installing DDP, the corresponding Deckhouse modules must be enabled in the cluster for each service with `mode: managed`:

| DDP setting | Module that must be enabled |
|---|---|
| `postgres.mode: managed` | `managed-postgres` |
| `redis.mode: managed` | `managed-valkey` |
| `clickhouse.mode: managed` | `managed-clickhouse` |

With `redis.mode: managed`, the `managed-valkey` module version 0.1.6 or later is required. DDP connects to Valkey as the `ddp` user; the module generates the password and stores it in the `redis-credentials` Secret in the `d8-development-platform` namespace.

For the demo configuration below, enable all three modules using ModuleConfig:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: managed-postgres
spec:
  enabled: true
---
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: managed-valkey
spec:
  enabled: true
---
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: managed-clickhouse
spec:
  enabled: true
```

If a module is not enabled, the portal is not installed, and the `development-platform` module status shows an error with the name of the module to enable.

Minimum portal configuration for a demo installation:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io
    security:
      secretKey: "16charssecretkey"
    postgres:
      mode: managed
    redis:
      mode: managed
    clickhouse:
      mode: managed
```

With `mode: managed`, the connection host and port are resolved automatically.
The `host`, `port` and `image` parameters are not used in this mode.

Set instance resources in `instance.cpu`, `instance.memory` and `instance.persistentVolumeClaim`.
Select the class with `instance.className`, `default` by default; it defines allowed sizes and placement of the instance.
Specify memory and disk sizes as Kubernetes quantities, such as `512Mi` or `20Gi`.

Example of PostgreSQL resource settings:

```yaml
    postgres:
      mode: managed
      instance:
        className: default
        cpu:
          cores: 2
          coreFraction: 50              # For redis and clickhouse, a string, for example "50%".
        memory:
          size: 2Gi
        persistentVolumeClaim:
          size: 20Gi
          storageClassName: replicated  # Optional, the cluster default StorageClass is used by default.
```

The disk size can only be increased, and only if the StorageClass supports volume expansion.

## Installation with external instances

For production, connect existing PostgreSQL, Redis and ClickHouse instances with `mode: external`.
The sections are independent: the other services can explicitly use `internal` or `managed`.

### Connecting external PostgreSQL

To use an external PostgreSQL instance, specify connection parameters in the `postgres` section:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io
    security:
      secretKey: "16charssecretkey"
    postgres:
      mode: external
      host: postgres.example.com  # PostgreSQL server hostname or IP address.
      port: 5432                  # PostgreSQL port (default 5432).
      database: ddp               # Database name.
      username: ddp_user          # Connection username.
      password: secure_password   # Connection password.
    redis:
      mode: internal
    clickhouse:
      mode: internal
```

#### pg_trgm extension

The portal requires the PostgreSQL `pg_trgm` extension.
If you use an external PostgreSQL instance, enable it before starting DDP: connect to the database as a user with permission to create extensions and run:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

If you deploy the built-in or managed PostgreSQL instance, the extension is created automatically.

### Connecting external Redis

To use an external Redis instance, specify connection parameters in the `redis` section:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io
    security:
      secretKey: "16charssecretkey"
    redis:
      mode: external
      host: redis.example.com       # Redis server hostname or IP address.
      port: 6379                    # Redis port (default 6379).
      database: "0"                 # Redis database index (default "0").
      password: redis_password      # Connection password (optional; leave empty if Redis has no password).
    postgres:
      mode: internal
    clickhouse:
      mode: internal
```

If Redis uses ACL users, set the username in the `redis.username` parameter. With an empty value, DDP authenticates with the password only.

### Full example with external instances

Example configuration with connections to external PostgreSQL, Redis and ClickHouse instances:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io
    security:
      secretKey: "16charssecretkey"
    postgres:
      mode: external
      host: postgres.production.example.com
      port: 5432
      database: ddp
      username: ddp_user
      password: secure_postgres_password
    redis:
      mode: external
      host: redis.production.example.com
      port: 6379
      database: "0"
      password: secure_redis_password
    clickhouse:
      mode: external
      host: clickhouse.production.example.com
      port: 9000
      database: ddp
      username: ddp_user
      password: secure_clickhouse_password
```

## Installation with internal instances

With `mode: internal`, the module deploys PostgreSQL, Redis and ClickHouse in the cluster.
Enable `development-platform` with [ModuleConfig](/products/kubernetes-platform/documentation/v1/reference/api/cr.html#moduleconfig), specifying the super administrator, the encryption key and the mode of each service:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io # Superadministrator email with full access to portal configuration. Can be changed at any time.
    security:
      secretKey: "16charssecretkey" # Encryption key: 16, 24 or 32 printable ASCII characters. Replace the example with your own key.
    postgres:
      mode: internal
    redis:
      mode: internal
    clickhouse:
      mode: internal
```

The web interface address is built from the `publicDomainTemplate` cluster parameter and the `domain.prefix` module parameter, `ddp` by default.
For example, with `publicDomainTemplate: "%s.example.com"`, the web interface is available at `https://ddp.example.com`.
To use another domain, set `domain.useCustomDomain` and `domain.customDomain`.
All parameters are listed in the [module settings](/modules/development-platform/configuration.html).

This option deploys services through the module itself, without managed operators.

### Configuring internal instances (optional)

To pull internal PostgreSQL, Redis and ClickHouse images from a private registry, create a Secret of the `kubernetes.io/dockerconfigjson` type with the registry credentials in the `d8-development-platform` namespace.
Set the images in the `image` parameters and the Secret name in `additionalImagePullSecrets`:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io
    security:
      secretKey: "16charssecretkey"
    postgres:
      mode: internal
      image: registry.example.com/postgres:16.3  # PostgreSQL image from a private container registry.
    redis:
      mode: internal
      image: registry.example.com/redis:7.4.0    # Redis image from a private container registry.
    clickhouse:
      mode: internal
      image: registry.example.com/clickhouse/clickhouse-server:24.3
    additionalImagePullSecrets:
      - "custom-registry-secret"                 # (optional) additional secrets for private container registry access.
```

## ClickHouse

ClickHouse stores historical data, such as the history of entity property changes.
Select `internal` for an instance deployed by the module, `managed` to create an instance with the `managed-clickhouse` module, or `external` to connect to an existing server.
Use an external instance for production.

### Internal ClickHouse instance

To deploy ClickHouse as part of the module, explicitly set `clickhouse.mode: internal`.
The following example pulls the image from a private registry; first create the `custom-registry-secret` Secret with the registry credentials in the `d8-development-platform` namespace.
The `host` parameter is not used in this mode:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io
    security:
      secretKey: "16charssecretkey"
    clickhouse:
      mode: internal
      database: ddp                  # Database name, created on the first server start.
      username: default              # Connection username.
      password: clickhouse_password  # Password the in-cluster server is created with.
      image: registry.example.com/clickhouse/clickhouse-server:24.3  # (optional) image from a private container registry.
    postgres:
      mode: internal
    redis:
      mode: internal
    additionalImagePullSecrets:
      - custom-registry-secret
```

In this mode, the portal deploys one ClickHouse instance with `10Gi` of persistent storage.
The database is created on the first start with an empty volume.
After restoring data from a backup or changing the `database` parameter, create the database manually as described [below](#creating-the-clickhouse-database).

### External ClickHouse instance

To use an external ClickHouse instance, set `mode: external` and the connection parameters:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 2
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io
    security:
      secretKey: "16charssecretkey"
    clickhouse:
      mode: external
      host: clickhouse.example.com  # ClickHouse server hostname or IP address.
      port: 9000                    # ClickHouse native protocol port (default 9000).
      database: ddp                 # Database name.
      username: ddp_user            # Connection username.
      password: secure_password     # Connection password.
    postgres:
      mode: internal
    redis:
      mode: internal
```

{{< alert level="warning" >}}
Create the database before connecting an external instance: the portal applies schema migrations but does not create the database itself.
{{< /alert >}}

### Creating the ClickHouse database

With `mode: managed`, the module creates the database automatically.

When connecting an external instance, create the database before starting DDP.
Replace `ddp` with the value of `clickhouse.database`:

```sql
CREATE DATABASE IF NOT EXISTS ddp;
```

With `mode: internal`, the database is created on the first start with an empty volume.
If the volume already contains data or you change the `database` parameter, create the database manually.
Replace `ddp` with the database name:

```shell
d8 k -n d8-development-platform exec clickhouse-0 -c clickhouse -- \
  sh -c 'clickhouse-client --user "$CLICKHOUSE_USER" --password "$CLICKHOUSE_PASSWORD" --query "CREATE DATABASE IF NOT EXISTS ddp"'
```

### Running without ClickHouse

To run the portal without ClickHouse, set `mode: external` and leave the `host` parameter empty:

```yaml
    clickhouse:
      mode: external
      host: ""
```

In this case, the features that depend on ClickHouse are unavailable: for example, the history of entity property changes is not saved.

### Data delivery to ClickHouse

The portal workers deliver data from PostgreSQL to ClickHouse and clean it up afterwards, so at least one running worker is required to transfer data.
Delivery parameters are set in the `clickhouse.replication` section, the property history retention period and the cleanup interval are set in the `clickhouse.propertyHistory` section.

## Changing the encryption key

DDP encrypts stored credentials with the key from the `security.secretKey` parameter.
Replacing the key without re-encryption makes stored data unavailable.

To change the key:

1. In the DDP web interface, open "Administration" → "Credentials" and click "Rotate encryption key".
   The button is available to users with the `rotate:encryption-key` permission.
1. Enter the current key in "Old encryption key" and the new key in "New encryption key", then click "Re-encrypt".
1. Check that the results table has no records in the "Failed" column.
1. Set the new key in the `security.secretKey` parameter of ModuleConfig `development-platform`.
   The DDP pods restart automatically.

Stored credentials are unavailable between re-encryption and the pod restart with the new key.
Changing the key invalidates current user sessions.
The full procedure and operation results are described in the [key rotation guide](security/encryption-key-rotation/).
