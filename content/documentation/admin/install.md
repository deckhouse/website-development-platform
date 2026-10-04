---
title: Installation
description: Install Deckhouse Development Portal with internal, external, or managed PostgreSQL, Redis, and ClickHouse instances.
weight: 11
---

Deckhouse Development Portal can be installed in three ways: with [external PostgreSQL and Redis instances](#installation-with-external-instances) (connecting to databases already deployed outside the cluster), with [internal instances](#installation-with-internal-instances) (deploying PostgreSQL and Redis inside the cluster), or with [managed instances](#installation-with-managed-instances) (creating databases with Deckhouse modules). External instances are recommended for production; internal instances are suitable for testing and pilot use.

The portal also uses [ClickHouse](#clickhouse), a storage for large volumes of data. By default, ClickHouse is deployed inside the cluster.

## Installation with internal instances

To install Deckhouse Development Portal, enable the `development-platform` module in your Kubernetes cluster running on Deckhouse Platform. You can use [ModuleConfig](/products/kubernetes-platform/documentation/v1/reference/api/cr.html#moduleconfig) with minimal settings:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 1
  settings:
    rbac:
      superAdminEmail: admin@deckhouse.io # Superadministrator email with full access to portal configuration. Can be changed at any time.
    security:
      secretKey: "16charssecretkey" # Secret key for encrypting private data. If changed, API access tokens will need to be regenerated and users will need to re-enter their credentials.
```

After installation, the Deckhouse Development Portal web UI will be available at `https://ddp.<your domain>`.

When you do not specify `postgres` and `redis` sections, the portal deploys internal PostgreSQL and Redis instances inside the cluster. This scenario is not recommended for production and is suitable only for testing and pilot use; for production, use [external instances](#installation-with-external-instances).

### Configuring internal instances (optional)

If you use internal instances, you can explicitly set `mode: internal` and specify images from a private container registry:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 1
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
    additionalImagePullSecrets:
      - "custom-registry-secret"                 # (optional) additional secrets for private container registry access.
```

## Installation with external instances

This installation option is recommended for production: the portal connects to PostgreSQL and Redis deployed outside the cluster, which improves resilience and simplifies backup and scaling of databases.

### Connecting external PostgreSQL

To use an external PostgreSQL instance, specify connection parameters in the `postgres` section:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 1
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
```

#### pg_trgm extension

The portal requires the PostgreSQL `pg_trgm` extension. If you use an external PostgreSQL instance, enable it before starting DDP: connect to the database as a user with permission to create extensions and run:

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
  version: 1
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
```

### Full example with external instances

Example configuration with external PostgreSQL and Redis:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 1
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
```

## ClickHouse

ClickHouse is a storage for large volumes of data, such as the history of entity property changes.

You can deploy ClickHouse inside the cluster as part of the module or connect an external instance. By default (`clickhouse.mode: internal`), the portal deploys ClickHouse inside the cluster. For production, use an external instance.

### Internal ClickHouse instance

The `internal` mode is used by default. To change the parameters of the internal instance, set them in the `clickhouse` section. The `host` parameter is not used in this mode:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 1
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
```

In this mode, the portal deploys a single ClickHouse instance with persistent storage (a PersistentVolumeClaim of `10Gi`) and creates the database on the first start.

### External ClickHouse instance

To use an external ClickHouse instance, set `mode: external` and the connection parameters:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 1
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
```

{{< alert level="warning" >}}
Create the database before connecting an external instance: the portal applies schema migrations but does not create the database itself.
{{< /alert >}}

### Running without ClickHouse

To run the portal without ClickHouse, set `mode: external` and leave the `host` parameter empty:

```yaml
    clickhouse:
      mode: external
      host: ""
```

In this case, the features that depend on ClickHouse are unavailable: for example, the history of entity property changes is not saved.

### Data delivery to ClickHouse

The portal workers deliver data from PostgreSQL to ClickHouse and clean it up afterwards, so at least one running worker is required to transfer data. Delivery parameters are set in the `clickhouse.replication` section, the retention period of the property history and the cleanup interval are set in the `clickhouse.propertyHistory` section.

## Installation with managed instances

In the `managed` mode, the portal creates PostgreSQL, Valkey (Redis-compatible) and ClickHouse instances using the Deckhouse modules `managed-postgres`, `managed-valkey` and `managed-clickhouse`. The mode is set separately for each instance, and the portal creates the databases automatically. Enable the required modules before installing the portal:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: managed-postgres  # The same for managed-valkey and managed-clickhouse.
spec:
  enabled: true
```

If a module is not enabled, the portal is not installed, and the `development-platform` module status shows an error with the name of the module to enable.

Example of the portal configuration:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  enabled: true
  version: 1
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

Instance resources are set in the `instance` parameter of each instance. Its structure matches `spec.instance` of the managed module resources; the allowed values are limited by the class specified in `className` (`default` by default). For example:

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
