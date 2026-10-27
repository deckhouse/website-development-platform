---
title: Установка
description: Установка Deckhouse Development Portal с внутренними, внешними или managed-экземплярами PostgreSQL, Redis и ClickHouse.
weight: 11
---

Deckhouse Development Portal (DDP, портал) устанавливается модулем `development-platform` в кластере Deckhouse Platform.
Перед установкой проверьте [требования к вычислительным ресурсам](resource-requirements/).

Для демо рекомендуется установка с [managed-экземплярами](#установка-с-managed-экземплярами), для production — с [внешними экземплярами](#установка-с-внешними-экземплярами).
[Внутренние экземпляры](#установка-с-внутренними-экземплярами) доступны как отдельный вариант установки.

Режим `managed`, `external` или `internal` задаётся для PostgreSQL, Redis и ClickHouse отдельно, и режимы можно сочетать.

Параметры `postgres.mode`, `redis.mode` и `clickhouse.mode` обязательны и не имеют значения по умолчанию.
Без них установка и обновление модуля завершаются ошибкой проверки настроек.

ClickHouse хранит исторические данные портала.
Чтобы работать без функциональности, которой нужен ClickHouse, явно укажите `clickhouse.mode: external` и оставьте `clickhouse.host` пустым, как описано [ниже](#работа-без-clickhouse).

## Установка с managed-экземплярами

В режиме `managed` модуль создаёт экземпляры PostgreSQL, Valkey (совместим с Redis) и ClickHouse.
До установки DDP в кластере должны быть включены соответствующие модули Deckhouse для каждого сервиса, у которого выбран `mode: managed`:

| Параметр DDP | Модуль, который должен быть включён |
|---|---|
| `postgres.mode: managed` | `managed-postgres` |
| `redis.mode: managed` | `managed-valkey` |
| `clickhouse.mode: managed` | `managed-clickhouse` |

Для демо-конфигурации ниже включите все три модуля с помощью ModuleConfig:

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

Если модуль не включён, портал не установится, а в статусе модуля `development-platform` появится ошибка с названием модуля, который не включён.

Минимальная конфигурация портала для демо-установки:

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

В режиме `managed` адрес и порт подключения определяются автоматически.
Параметры `host`, `port` и `image` для этого режима не используются.

Ресурсы экземпляра задаются в `instance.cpu`, `instance.memory` и `instance.persistentVolumeClaim`.
Класс выбирается параметром `instance.className`, по умолчанию `default`; он определяет допустимые размеры и размещение экземпляра.
Размеры памяти и диска задавайте в формате Kubernetes Quantity, например `512Mi` или `20Gi`.

Пример настройки ресурсов PostgreSQL:

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

## Установка с внешними экземплярами

Для production подключите уже развёрнутые PostgreSQL, Redis и ClickHouse в режиме `external`.
Секции независимы, поэтому для остальных сервисов можно явно выбрать `internal` или `managed`.

### Подключение внешнего PostgreSQL

Для использования внешнего экземпляра PostgreSQL укажите параметры подключения в секции `postgres`:

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
      host: postgres.example.com  # Имя хоста или IP-адрес сервера PostgreSQL.
      port: 5432                  # Порт PostgreSQL (по умолчанию 5432).
      database: ddp               # Название базы данных.
      username: ddp_user          # Имя пользователя для подключения.
      password: secure_password   # Пароль для подключения.
    redis:
      mode: internal
    clickhouse:
      mode: internal
```

#### Расширение pg_trgm

Порталу требуется расширение PostgreSQL `pg_trgm`.
Если используется внешний PostgreSQL, включите его до запуска DDP — подключитесь к базе данных под пользователем с правами на создание расширений и выполните:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

При установке с внутренними и managed-экземплярами расширение создаётся автоматически.

### Подключение внешнего Redis

Для использования внешнего экземпляра Redis укажите параметры подключения в секции `redis`:

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
      host: redis.example.com       # Имя хоста или IP-адрес сервера Redis.
      port: 6379                    # Порт Redis (по умолчанию 6379).
      database: "0"                 # Индекс базы данных Redis (по умолчанию "0").
      password: redis_password      # Пароль для подключения (необязательно; если Redis без пароля — оставить пустым).
    postgres:
      mode: internal
    clickhouse:
      mode: internal
```

### Полный пример с внешними экземплярами

Пример конфигурации с подключением к внешним экземплярам PostgreSQL, Redis и ClickHouse:

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

## Установка с внутренними экземплярами

В режиме `internal` модуль разворачивает PostgreSQL, Redis и ClickHouse в кластере.
Включите `development-platform` с помощью [ModuleConfig](/products/kubernetes-platform/documentation/v1/reference/api/cr.html#moduleconfig), указав суперадминистратора, ключ шифрования и режим каждого сервиса:

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
      superAdminEmail: admin@deckhouse.io # Email суперадминистратора, который будет иметь полный доступ к конфигурации портала. Может быть изменён в любой момент.
    security:
      secretKey: "16charssecretkey" # Ключ шифрования: 16, 24 или 32 печатных ASCII-символа. Замените пример своим ключом.
    postgres:
      mode: internal
    redis:
      mode: internal
    clickhouse:
      mode: internal
```

Адрес веб-интерфейса формируется из параметра кластера `publicDomainTemplate` и параметра модуля `domain.prefix`, по умолчанию `ddp`.
Например, при `publicDomainTemplate: "%s.example.com"` веб-интерфейс доступен по адресу `https://ddp.example.com`.
Для другого домена задайте `domain.useCustomDomain` и `domain.customDomain`.
Все параметры приведены [в настройках модуля](/modules/development-platform/configuration.html).

Этот вариант разворачивает сервисы самим модулем, без managed-операторов.

### Настройка внутренних экземпляров (опционально)

Чтобы загрузить образы внутренних PostgreSQL, Redis и ClickHouse из приватного хранилища, создайте Secret типа `kubernetes.io/dockerconfigjson` с учётными данными хранилища в неймспейсе `d8-development-platform`.
Укажите образы в параметрах `image`, а название Secret в `additionalImagePullSecrets`:

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
      image: registry.example.com/postgres:16.3  # Образ PostgreSQL из приватного хранилища образов контейнеров.
    redis:
      mode: internal
      image: registry.example.com/redis:7.4.0    # Образ Redis из приватного хранилища образов контейнеров.
    clickhouse:
      mode: internal
      image: registry.example.com/clickhouse/clickhouse-server:24.3
    additionalImagePullSecrets:
      - "custom-registry-secret"                 # (опционально) дополнительные секреты для доступа к приватному хранилищу образов контейнеров.
```

## ClickHouse

ClickHouse хранит исторические данные, например [историю изменений параметров сущностей](../user/catalog/#история-параметров).
Выберите `internal` для экземпляра в составе модуля, `managed` для создания через модуль `managed-clickhouse` или `external` для подключения существующего сервера.
Для промышленной эксплуатации используйте внешний экземпляр.

### Внутренний экземпляр ClickHouse

Чтобы развернуть ClickHouse в составе модуля, явно задайте `clickhouse.mode: internal`.
В следующем примере образ загружается из приватного хранилища; предварительно создайте Secret `custom-registry-secret` с учётными данными хранилища в неймспейсе `d8-development-platform`.
Параметр `host` в этом режиме не используется:

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
      database: ddp                  # Название базы данных, создаётся при первом запуске сервера.
      username: default              # Имя пользователя для подключения.
      password: clickhouse_password  # Пароль, с которым создаётся сервер в кластере.
      image: registry.example.com/clickhouse/clickhouse-server:24.3  # (опционально) образ из приватного хранилища образов контейнеров.
    postgres:
      mode: internal
    redis:
      mode: internal
    additionalImagePullSecrets:
      - custom-registry-secret
```

В этом режиме портал разворачивает один экземпляр ClickHouse с постоянным хранилищем размером `10Gi`.
База данных создаётся при первом запуске на пустом томе.
После восстановления данных из резервной копии или изменения параметра `database` создайте базу данных вручную, как описано [ниже](#создание-базы-данных-clickhouse).

### Внешний экземпляр ClickHouse

Для использования внешнего экземпляра ClickHouse укажите `mode: external` и параметры подключения:

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
      host: clickhouse.example.com  # Имя хоста или IP-адрес сервера ClickHouse.
      port: 9000                    # Порт нативного протокола ClickHouse (по умолчанию 9000).
      database: ddp                 # Название базы данных.
      username: ddp_user            # Имя пользователя для подключения.
      password: secure_password     # Пароль для подключения.
    postgres:
      mode: internal
    redis:
      mode: internal
```

{{< alert level="warning" >}}
Создайте базу данных до подключения внешнего экземпляра, поскольку портал применяет миграции схемы, но саму базу данных не создаёт.
{{< /alert >}}

### Создание базы данных ClickHouse

При `mode: managed` модуль создаёт базу данных автоматически.

При подключении внешнего экземпляра создайте базу данных до запуска DDP.
Замените `ddp` значением параметра `clickhouse.database`:

```sql
CREATE DATABASE IF NOT EXISTS ddp;
```

При `mode: internal` база создаётся при первом запуске на пустом томе.
Если том уже содержит данные или изменён параметр `database`, создайте базу вручную.
Замените `ddp` названием базы:

```shell
d8 k -n d8-development-platform exec clickhouse-0 -c clickhouse -- \
  sh -c 'clickhouse-client --user "$CLICKHOUSE_USER" --password "$CLICKHOUSE_PASSWORD" --query "CREATE DATABASE IF NOT EXISTS ddp"'
```

### Работа без ClickHouse

Чтобы портал работал без ClickHouse, укажите `mode: external` и оставьте параметр `host` пустым:

```yaml
    clickhouse:
      mode: external
      host: ""
```

В этом случае возможности, которые зависят от ClickHouse, недоступны.
Например, не сохраняется история изменений параметров сущностей.

### Доставка данных в ClickHouse

Доставку данных из PostgreSQL в ClickHouse и последующую очистку выполняют воркеры портала, поэтому для переноса данных нужен хотя бы один запущенный воркер.
Параметры доставки задаются в секции `clickhouse.replication`, [срок хранения истории параметров](../user/catalog/#хранение-истории) и периодичность очистки — в секции `clickhouse.propertyHistory`.
Подробнее о воркерах — в разделе [«Воркеры»](architecture/workers/).

## Смена ключа шифрования

DDP шифрует сохранённые учётные данные ключом из параметра `security.secretKey`.
Если заменить ключ без перешифрования, сохранённые данные станут недоступны.

Чтобы сменить ключ:

1. В веб-интерфейсе DDP откройте «Администрирование» → «Учётные данные» и нажмите «Ротация ключа шифрования».
   Кнопка доступна пользователям с правом `rotate:encryption-key`.
1. Введите текущий ключ в поле «Старый ключ шифрования», новый в поле «Новый ключ шифрования» и нажмите «Перешифровать».
1. Проверьте, что в таблице результатов нет записей в столбце «Неуспешно».
1. Укажите новый ключ в параметре `security.secretKey` в ModuleConfig `development-platform`.
   Поды DDP перезапустятся автоматически.

Между перешифрованием и перезапуском подов с новым ключом сохранённые учётные данные недоступны.
Смена ключа делает текущие пользовательские сеансы недействительными.
Подробный порядок и результаты операции описаны [в руководстве по ротации ключа шифрования](security/encryption-key-rotation/).
