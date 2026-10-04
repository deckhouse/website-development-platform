---
title: Установка
description: Установка Deckhouse Development Portal с внутренними, внешними или managed-экземплярами PostgreSQL, Redis и ClickHouse.
weight: 11
---

Deckhouse Development Portal (DDP, портал) можно установить тремя способами: с [внешними экземплярами](#установка-с-внешними-экземплярами) PostgreSQL и Redis (подключение к уже развёрнутым базам данных вне кластера), с [внутренними экземплярами](#установка-с-внутренними-экземплярами) (развёртывание PostgreSQL и Redis внутри кластера) или с [managed-инстансами](#установка-с-managed-инстансами) (создание баз данных с помощью модулей Deckhouse). Внешние экземпляры рекомендуются для production, внутренние подходят для тестов и пилотной эксплуатации.

Дополнительно портал может использовать [ClickHouse](#clickhouse) — хранилище для больших объёмов данных. По умолчанию ClickHouse развёртывается внутри кластера.

## Установка с внутренними экземплярами

Для установки DDP включите модуль `development-platform` в вашем кластере Kubernetes под управлением Deckhouse Platform. Для этого можно использовать [ModuleConfig](/products/kubernetes-platform/documentation/v1/reference/api/cr.html#moduleconfig) с минимальным количеством настроек:

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
      superAdminEmail: admin@deckhouse.io # Email суперадминистратора, который будет иметь полный доступ к конфигурации портала. Может быть изменён в любой момент.
    security:
      secretKey: "16charssecretkey" # Секретный ключ для шифрования приватных данных. При изменении потребуется перегенерация токенов доступа к API портала и повторное заполнение учётных данных пользователями.
```

После установки веб-интерфейс DDP будет доступен по адресу `https://ddp.<ваш домен>`.

При развёртывании без указания секций `postgres` и `redis` портал разворачивает внутренние экземпляры PostgreSQL и Redis внутри кластера. Такой сценарий не рекомендуется для production и подходит только для тестов и пилотной эксплуатации; для промышленной эксплуатации используйте [внешние экземпляры](#установка-с-внешними-экземплярами).

### Настройка внутренних экземпляров (опционально)

Если вы используете внутренние экземпляры, можно явно указать `mode: internal` и задать образы из приватного хранилища образов контейнеров:

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
      image: registry.example.com/postgres:16.3  # Образ PostgreSQL из приватного хранилища образов контейнеров.
    redis:
      mode: internal
      image: registry.example.com/redis:7.4.0    # Образ Redis из приватного хранилища образов контейнеров.
    additionalImagePullSecrets:
      - "custom-registry-secret"                 # (опционально) дополнительные секреты для доступа к приватному хранилищу образов контейнеров.
```

## Установка с внешними экземплярами

Этот вариант установки рекомендуется для production: портал подключается к уже развёрнутым PostgreSQL и Redis вне кластера, что обеспечивает отказоустойчивость и упрощает резервное копирование и масштабирование баз данных.

### Подключение внешнего PostgreSQL

Для использования внешнего экземпляра PostgreSQL необходимо указать параметры подключения в секции `postgres`:

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
      host: postgres.example.com  # Имя хоста или IP-адрес сервера PostgreSQL.
      port: 5432                  # Порт PostgreSQL (по умолчанию 5432).
      database: ddp               # Название базы данных.
      username: ddp_user          # Имя пользователя для подключения.
      password: secure_password   # Пароль для подключения.
```

#### Расширение pg_trgm

Порталу требуется расширение PostgreSQL `pg_trgm`. Если используется внешний PostgreSQL, включите его до запуска DDP — подключитесь к базе данных под пользователем с правами на создание расширений и выполните:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

При установке с внутренними и managed-экземплярами расширение создаётся автоматически.

### Подключение внешнего Redis

Для использования внешнего экземпляра Redis необходимо указать параметры подключения в секции `redis`:

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
      host: redis.example.com       # Имя хоста или IP-адрес сервера Redis.
      port: 6379                    # Порт Redis (по умолчанию 6379).
      database: "0"                 # Индекс базы данных Redis (по умолчанию "0").
      password: redis_password      # Пароль для подключения (необязательно; если Redis без пароля — оставить пустым).
```

### Полный пример с внешними экземплярами

Пример конфигурации с подключением к внешним экземплярам PostgreSQL и Redis:

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

ClickHouse — хранилище для больших объёмов данных, например [истории изменений параметров сущностей](../../user/catalog/#история-параметров). ClickHouse можно развернуть внутри кластера в составе модуля либо подключить внешний экземпляр. По умолчанию (`clickhouse.mode: internal`) портал развёртывает ClickHouse внутри кластера. Для промышленной эксплуатации используйте внешний экземпляр.

### Внутренний экземпляр ClickHouse

Режим `internal` используется по умолчанию. Чтобы изменить параметры внутреннего экземпляра, задайте их в секции `clickhouse`. Параметр `host` в этом режиме не используется:

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
      database: ddp                  # Название базы данных, создаётся при первом запуске сервера.
      username: default              # Имя пользователя для подключения.
      password: clickhouse_password  # Пароль, с которым создаётся сервер в кластере.
      image: registry.example.com/clickhouse/clickhouse-server:24.3  # (опционально) образ из приватного хранилища образов контейнеров.
```

В этом режиме портал разворачивает один экземпляр ClickHouse с постоянным хранилищем (PersistentVolumeClaim размером `10Gi`) и сам создаёт базу данных при первом запуске.

### Внешний экземпляр ClickHouse

Для использования внешнего экземпляра ClickHouse укажите `mode: external` и параметры подключения:

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
      host: clickhouse.example.com  # Имя хоста или IP-адрес сервера ClickHouse.
      port: 9000                    # Порт нативного протокола ClickHouse (по умолчанию 9000).
      database: ddp                 # Название базы данных.
      username: ddp_user            # Имя пользователя для подключения.
      password: secure_password     # Пароль для подключения.
```

{{< alert level="warning" >}}
Создайте базу данных до подключения внешнего экземпляра: портал применяет миграции схемы, но саму базу данных не создаёт.
{{< /alert >}}

### Работа без ClickHouse

Чтобы портал работал без ClickHouse, укажите `mode: external` и оставьте параметр `host` пустым:

```yaml
    clickhouse:
      mode: external
      host: ""
```

В этом случае возможности, которые зависят от ClickHouse, недоступны: например, не сохраняется история изменений параметров сущностей.

### Доставка данных в ClickHouse

Доставку данных из PostgreSQL в ClickHouse и последующую очистку выполняют воркеры портала, поэтому для переноса данных нужен хотя бы один запущенный воркер. Параметры доставки задаются в секции `clickhouse.replication`, [срок хранения истории параметров](../../user/catalog/#хранение-истории) и периодичность очистки — в секции `clickhouse.propertyHistory`. Подробнее о воркерах — в разделе [«Воркеры»](../architecture/workers/).

## Установка с managed-инстансами

В режиме `managed` портал создаёт экземпляры PostgreSQL, Valkey (совместим с Redis) и ClickHouse с помощью модулей Deckhouse `managed-postgres`, `managed-valkey` и `managed-clickhouse`. Режим задаётся для каждого инстанса отдельно, базы данных портал создаёт автоматически. Включите нужные модули до установки портала:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: managed-postgres  # Так же для managed-valkey и managed-clickhouse.
spec:
  enabled: true
```

Если модуль не включён, портал не установится, а в статусе модуля `development-platform` появится ошибка с названием модуля, который нужно включить.

Пример конфигурации портала:

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

Ресурсы экземпляра задаются в параметре `instance` каждого инстанса. Структура совпадает с `spec.instance` ресурсов managed-модулей, допустимые значения ограничивает класс, указанный в `className` (по умолчанию — `default`). Например:

```yaml
    postgres:
      mode: managed
      instance:
        className: default
        cpu:
          cores: 2
          coreFraction: 50              # Для redis и clickhouse — строка, например "50%".
        memory:
          size: 2Gi
        persistentVolumeClaim:
          size: 20Gi
          storageClassName: replicated  # Необязательно, по умолчанию — StorageClass кластера.
```

Размер диска можно только увеличить, и только если StorageClass поддерживает расширение томов.
