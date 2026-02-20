---
title: Installation
weight: 11
---

## Enabling the module

To install Deckhouse Development Platform, enable the `development-platform` module in your Kubernetes cluster running on Deckhouse Kubernetes Platform. You can use [ModuleConfig](../../../../kubernetes-platform/documentation/v1/reference/api/cr.html#moduleconfig) with minimal settings:

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
      superAdminEmail: admin@deckhouse.io # Super administrator email with full access to platform configuration. Can be changed at any time
    security:
      secretKey: "16charssecretkey" # Secret key for encrypting private data. If changed, API access tokens will need to be regenerated and users will need to re-enter their credentials
```

After installation, the Deckhouse Development Platform web UI will be available at `https://ddp.<your domain>`.

With this configuration, Redis and PostgreSQL will be deployed into the cluster. This scenario is not recommended for production and is suitable only for testing and pilot use. For production, use dedicated PostgreSQL and Redis instances (see sections below).

## Connecting external instances

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
      host: postgres.example.com  # PostgreSQL server hostname or IP address
      port: 5432                  # PostgreSQL port (default 5432)
      database: ddp               # Database name
      username: ddp_user          # Connection username
      password: secure_password   # Connection password
```

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
      host: redis.example.com       # Redis server hostname or IP address
      port: 6379                    # Redis port (default 6379)
      database: "0"                 # Redis database index (default "0")
      password: redis_password      # Connection password (optional; leave empty if Redis has no password)
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

## Installation with internal instances

If you do not connect external databases (as in the sections above), the platform can deploy PostgreSQL and Redis inside the cluster. Example configuration with internal instances and a private Docker registry:

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
      image: registry.example.com/postgres:16.3  # PostgreSQL image from private registry
    redis:
      mode: internal
      image: registry.example.com/redis:7.4.0    # Redis image from private registry
    additionalImagePullSecrets:
      - "custom-registry-secret"                 # (optional) additional secrets for private registry access
```
