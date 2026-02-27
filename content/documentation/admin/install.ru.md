---
title: Установка
weight: 11
---

## Включение модуля

Для установки Deckhouse Development Platform включите модуль `development-platform` в вашем Kubernetes-кластере под управлением Deckhouse Kubernetes Platform. Для этого можно использовать [ModuleConfig](../../../../kubernetes-platform/documentation/v1/reference/api/cr.html#moduleconfig) с минимальным количеством настроек:

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
      superAdminEmail: admin@deckhouse.io # Email супер-администратора, который будет иметь полный доступ к конфигурации платформы. Может быть изменен в любой момент
    security:
      secretKey: "16charssecretkey" # Секретный ключ для шифрования приватных данных. При изменении потребуется перегенерация токенов доступа к API платформы и повторное заполнение учетных данных пользователями
```

После установки веб-интерфейс Deckhouse Development Platform будет доступен по адресу `https://ddp.<ваш домен>`.

При развертывании в такой конфигурации в кластер будут установлены Redis и PostgreSQL. Такой сценарий не рекомендуется для production и подходит только для тестов и пилотной эксплуатации. В промышленной эксплуатации рекомендуется использовать выделенные экземпляры PostgreSQL и Redis (см. разделы ниже).

## Подключение внешних инстансов

### Подключение внешнего PostgreSQL

Для использования внешнего инстанса PostgreSQL необходимо указать параметры подключения в секции `postgres`:

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
      host: postgres.example.com  # Имя хоста или IP-адрес сервера PostgreSQL
      port: 5432                  # Порт PostgreSQL (по умолчанию 5432)
      database: ddp               # Название базы данных
      username: ddp_user          # Имя пользователя для подключения
      password: secure_password   # Пароль для подключения
```

### Подключение внешнего Redis

Для использования внешнего инстанса Redis необходимо указать параметры подключения в секции `redis`:

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
      host: redis.example.com       # Имя хоста или IP-адрес сервера Redis
      port: 6379                    # Порт Redis (по умолчанию 6379)
      database: "0"                 # Индекс базы данных Redis (по умолчанию "0")
      password: redis_password      # Пароль для подключения (необязательно; если Redis без пароля — оставить пустым)
```

### Полный пример с внешними инстансами

Пример конфигурации с подключением к внешним инстансам PostgreSQL и Redis:

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

## Установка с внутренними инстансами

Если вы не подключаете внешние базы (как в разделах выше), платформа может разворачивать PostgreSQL и Redis внутри кластера. Пример конфигурации с внутренними инстансами и использованием приватного Docker registry:

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
      image: registry.example.com/postgres:16.3  # Образ PostgreSQL из приватного registry
    redis:
      mode: internal
      image: registry.example.com/redis:7.4.0    # Образ Redis из приватного registry
    additionalImagePullSecrets:
      - "custom-registry-secret"                 # (опционально) дополнительные секреты для доступа к приватному registry
```
