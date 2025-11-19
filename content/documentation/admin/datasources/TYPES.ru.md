---
title: Типы источников данных
---

## DefectdojoProducts

Источник данных со встроенным backend **DefectdojoProducts** возвращает список продуктов в Defectdojo.

<a id="defectdojoproductsauth"></a>

### Аутентификация

Для работы источника данных необходим API v2 Key токен. Этот токен передается через конфигурацию Headers ([подробнее](#defectdojoproductsheaders)).

Подробнее о том, как получить API v2 Key токен можно узнать в [официальной документации](https://docs.defectdojo.com/en/api/api-v2-docs/).

### Спецификация ответа

Платформа выполняет GET-запрос по URL: `/api/v2/products`. Возвращаются все доступные значения. [Спецификация ответа](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).

### Конфигурация

* **URL** - URL DefectDojo в формате `https://example.com`.

<a id="defectdojoproductsheaders"></a>
**Headers:**

| Название      | Обязательность  | Описание                                                                |Пример        |
|---------------|-----------------|-------------------------------------------------------------------------|--------------|
| Authorization | **обязательно** | API v2 Key токен для авторизации в DefectDojo. [Подробнее](#defectdojoproductsauth)|Token <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## GitlabGroups

Источник данных со встроенным backend **GitlabGroups** возвращает список групп в GitLab.

<a id="gitlabgroupsauth"></a>

### Аутентификация

Для работы источника данных необходим GitLab token. Этот токен передается через конфигурацию Headers ([подробнее](#gitlabgroupsheaders)).

Подробнее о том, как получить GitLab token можно узнать в [официальной документации](https://docs.gitlab.com/api/rest/authentication/).

### Спецификация ответа

Платформа выполняет GET-запрос к API GitLab по URL: `/api/v4/groups`. Платформа возвращает все доступные значения. [Спецификация ответа](https://docs.gitlab.com/api/groups/#list-groups).

### Конфигурация

* **URL** - URL GitLab в формате `https://gitlab.com`, без части `/api/v4`.

<a id="gitlabgroupsheaders"></a>
**Headers:**

| Название      | Обязательность  | Описание                                                       |
|---------------|-----------------|----------------------------------------------------------------|
| Private-Token | **обязательно** | Токен для авторизации в GitLab. [Подробнее](#gitlabgroupsauth)|

**Параметры:**

Настраиваемые параметры отсутствуют.

## GitlabProjects

Источник данных со встроенным backend **GitlabProjects** возвращает список проектов в Gitlab.

<a id="gitlabprojectsauth"></a>

### Аутентификация

Для работы источника данных необходим GitLab token. Этот токен передается через конфигурацию Headers ([подробнее](#gitlabprojectsheaders)).

Подробнее о том, как получить GitLab token можно узнать в [официальной документации](https://docs.gitlab.com/api/rest/authentication/).

### Спецификация ответа

В зависимости от [конфигурации параметров](#gitlabprojectsparameters), платформа выполняет GET-запрос к API GitLab и возвращает соответствующую спецификацию.

Если значение параметра **all** равно `true`, выполняется GET-запрос по URL: `/api/v4/projects`. Платформа возвращает все доступные значения. [Спецификация ответа](https://docs.gitlab.com/api/projects/#list-all-projects).

Если значение параметра **all** равно `false`, выполняется GET-запрос по URL: `/api/v4/groups/:id/projects`. Платформа возвращает все доступные значения. [Спецификация ответа](https://docs.gitlab.com/api/projects/#list-all-projects).

Если значение параметра **tags** равно `true` платформа дополнительно получает git теги. Для получения git тегов, выполняется GET-запрос по URL: `/api/v4/projects/:id/repository/tags`. Платформа получает список всех git тегов и расширяет [спецификацию ответа](https://docs.gitlab.com/api/projects/#list-all-projects) полем `ddp_repository_tags`, которое соответствует [спецификации ответа list-project-repository-tags](https://docs.gitlab.com/api/tags/#list-project-repository-tags).

### Конфигурация

* **URL** - URL GitLab в формате `https://gitlab.com`, без части `/api/v4`.

<a id="gitlabprojectsheaders"></a>
**Headers:**

|Название     | Обязательность|Описание                                 |
|-------------|---------------|-----------------------------------------|
|Private-Token| **обязательно**|Токен для авторизации в GitLab. [Подробнее](#gitlabprojectsauth)|

<a id="gitlabprojectsparameters"></a>
**Параметры:**

|Название         |Обязательность                              |Описание                                                                                                                                                                                                                                                                                                                 |Возможные значения                                                            |По умолчанию                                                                  |
|-----------------|--------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|------------------------------------------------------------------------------|
|all              |опционально                                 |В явном виде указывает, что необходимо собирать репозитории всех групп, к которым есть доступ.                                                                                                                                                                                                                           |true, false                                                                   |false                                                                         |
|group_ids        |Обязательно, если **all** в значении `false`|Источник данных будет собирать проекты групп с указанным ID. ID групп указываются через запятую.                                                                                                                                                                                                                         |Пример: 1001,1002                                                             |\-                                                                            |
|tags             |опционально                                 |Платформа дополнительно получает список всех git тегов и расширяет [спецификацию ответа](https://docs.gitlab.com/api/projects/#list-all-projects) полем `ddp_repository_tags`, которое соответствует [спецификации ответа list-project-repository-tags](https://docs.gitlab.com/api/tags/#list-project-repository-tags). |true, false                                                                   |false                                                                         |
|include_subgroups|опционально                                 |Если указан параметр **group_ids**, то параметр **include_subgroups** определяет, собирать ли проекты подгрупп указанных групп.                                                                                                                                                                                          |true, false                                                                   |false                                                                         |
|tags_order_by    |опционально                                 |Соответствует спецификации [GitLab Tags API order_by](https://docs.gitlab.com/api/tags/#list-project-repository-tags)                                                                                                                                                                                                    |[документация](https://docs.gitlab.com/api/tags/#list-project-repository-tags)|[документация](https://docs.gitlab.com/api/tags/#list-project-repository-tags)|
|tags_sort        |опционально                                 |Соответствует спецификации [GitLab Tags API sort](https://docs.gitlab.com/api/tags/#list-project-repository-tags)                                                                                                                                                                                                        |[документация](https://docs.gitlab.com/api/tags/#list-project-repository-tags)|[документация](https://docs.gitlab.com/api/tags/#list-project-repository-tags)|
|tags_search      |опционально                                 |Соответствует спецификации [GitLab Tags API search](https://docs.gitlab.com/api/tags/#list-project-repository-tags)                                                                                                                                                                                                      |[документация](https://docs.gitlab.com/api/tags/#list-project-repository-tags)|[документация](https://docs.gitlab.com/api/tags/#list-project-repository-tags)|

## HarborArtifacts

Источник данных со встроенным backend **HarborArtifacts** собирает информацию о всех артефактах в Harbor.

<a id="harborartifactsauth"></a>

### Аутентификация

Аутентификация в Harbor осуществляется с помощью Basic Authentication. Для настройки аутентификации необходимо:

1. Сформировать строку username:password
2. Закодировать строку в Base64. Пример команды: `echo "username:password" | base64`
3. В конфигурации источника данных добавить заголовок `Authorization`. [Подробнее](#harborartifactsheaders).

### Спецификация ответа

Платформа получает список всех проектов и репозиториев, которые содержатся в этих проектах, затем по каждому из доступных проектов выполняется GET-запрос к API Harbor: `/api/v2.0/projects/{project_name}/repositories/{repository_name}/artifacts`. Спецификация ответа доступна в интерфейса Harbor ([подробнее](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/)).

### Конфигурация

* **URL** - URL Harbor в формате `https://example.com`

<a id="harborartifactsheaders"></a>
**Headers:**

|Название      |Обязательность     |Описание                                                           |Пример         |
|--------------|-------------------|-------------------------------------------------------------------|---------------|
|Authorization |**обязательно**    | Токен для авторизации в Harbor. [Подробнее](#harborartifactsauth) | Basic <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## HarborProjects

Источник данных со встроенным backend **HarborProjects** собирает информацию о всех проектах в Harbor.

<a id="harborprojectsauth"></a>

### Аутентификация

Аутентификация в Harbor осуществляется с помощью Basic Authentication. Для настройки аутентификации необходимо:

1. Сформировать строку username:password
2. Закодировать строку в Base64. Пример команды: `echo "username:password" | base64`
3. В конфигурации источника данных добавить заголовок `Authorization`. [Подробнее](#harborprojectsheaders).

### Спецификация ответа

Платформа получает список всех доступных проектов выполняя GET-запросы к API Harbor: `/api/v2.0/projects`. пецификация ответа доступна в интерфейса Harbor ([подробнее](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/)).

### Конфигурация

* **URL** - URL Harbor в формате `https://example.com`.

<a id="harborprojectsheaders"></a>
**Headers:**

|Название      |Обязательность    |Описание                                                           |Пример         |
|--------------|------------------|-------------------------------------------------------------------|---------------|
|Authorization |**обязательно**   | Токен для авторизации в Harbor. [Подробнее](#harborartifactssauth) | Basic <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## HarborRepositories

Источник данных со встроенным backend **HarborRepositories** собирает информацию о всех репозиториях в Harbor.

<a id="harborrepositoriesauth"></a>

### Аутентификация

Аутентификация в Harbor осуществляется с помощью Basic Authentication. Для настройки аутентификации необходимо:

1. Сформировать строку username:password
2. Закодировать строку в Base64. Пример команды: `echo "username:password" | base64`
3. В конфигурации источника данных добавить заголовок `Authorization`. [Подробнее](#harborrepositoriesheaders).

### Спецификация ответа

Платформа получает список всех проектов, затем получает список всех репозиториев в каждом из проектов выполняя GET-запросы к API Harbor: `/api/v2.0/projects/{project_name}/repositories`. Спецификация ответа доступна в интерфейса Harbor ([подробнее](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/)).

### Конфигурация

* **URL** - URL Harbor в формате `https://example.com`.

<a id="harborrepositoriesheaders"></a>
**Headers:**

|Название      |Обязательность    |Описание                                                               |Пример         |
|--------------|------------------|-----------------------------------------------------------------------|---------------|
|Authorization |**обязательно**   | Токен для авторизации в Harbor. [Подробнее](#harborrepositoriesauth) | Basic <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## HarborTags

Источник данных со встроенным backend **HarborTags** собирает информацию о всех тегах у артефактов в Harbor.

<a id="harbortagsauth"></a>

### Аутентификация

Аутентификация в Harbor осуществляется с помощью Basic Authentication. Для настройки аутентификации необходимо:

1. Сформировать строку username:password
2. Закодировать строку в Base64. Пример команды: `echo "username:password" | base64`
3. В конфигурации источника данных добавить заголовок `Authorization`. [Подробнее](#harbortagsheaders).

### Спецификация ответа

Платформа получает список всех проектов и репозиториев, которые содержатся в этих проектах, затем по каждому из доступных проектов выполняется GET-запрос к API Harbor: `/api/v2.0/projects/{project_name}/repositories/{repository_name}/artifacts`. Затем происходит сбор всех тегов по всем артефактам (поле `tags`) и результат возвращается в виде массива. Спецификация ответа доступна в интерфейса Harbor ([подробнее](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/)).

### Конфигурация

* **URL** - URL Harbor в формате `https://example.com`

<a id="harbortagsheaders"></a>
**Headers:**

|Название      |Обязательность     |Описание                                                      |Пример         |
|--------------|-------------------|--------------------------------------------------------------|---------------|
|Authorization |**обязательно**    | Токен для авторизации в Harbor. [Подробнее](#harbortagsauth) | Basic <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## HelmReleases

Источник данных со встроенным backend **HelmReleases** возвращает список всех HelmReleases в кластере Kubernetes.

<a id="helmreleasesauth"></a>

### Аутентификация

Платформа производит аутентификацию внутри кластера Kubernetes с помощью токена. Этот токен передается через конфигурацию Headers ([подробнее](#helmreleasesheaders)).

Подробнее про аутентификацию в Kubernetes описано в [официальной документации](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

### Спецификация ответа

Платформа возвращает все HelmReleases в кластере Kubernetes. Спецификация:

```json
[
  {
    "name": "string",              // Название
    "info": {
      "first_deployed": "string",  // Дата и время первого деплоя
      "last_deployed": "string",   // Дата и время последнего деплоя
      "deleted": "string",         // Дата и время удаления (может быть пустой строкой)
      "description": "string",     // Описание
      "status": "string",          // Статус
      "notes": "string"            // Заметки
    },
    "chart": {
      "metadata": {
        "name": "string",          // Название чарта
        "home": "string",          // Домашняя страница
        "sources": "array",        // Массив строк с URL-адресами источников
        "version": "string",       // Версия чарта
        "description": "string",   // Описание чарта
        "keywords": "array",       // Массив строк с ключевыми словами
        "maintainers": "array",    // Массив объектов с информацией о мейнтейнерах
        "icon": "string",          // URL иконки
        "apiVersion": "string",    // Версия API
        "appVersion": "string"     // Версия приложения
      },
      "templates": [               // Массив объектов с шаблонами
        {
          "name": "templates/NOTES.txt",
          "data": "..."
        },
        {
          "name": "templates/_helpers.tpl",
          "data": "..."
        }
        // ... другие шаблоны
      ],        
      "values": "object",          // Объект с доступными настройками Helm чарта
      "schema": "null",            // Схема (может быть null)
      "files": [                   // Массив объектов с файлами
        {
          "name": ".helmignore",
          "data": "..."
        },
        {
          "name": "LICENSE",
          "data": "..."
        }
        // ... другие файлы
      ]             
    },
    "config": {        // Текушая конфигурацию
      "caSecretName": "string",
      "cache": {
        "enabled": "boolean", 
        "expireHours": "integer"
      }
      // ... другие настройки
    },
    "manifest": "string", // Отрендеренные манифесты
    "version": "integer", // Версия helmrelease
    "namespace": "string" // Namespace, где развернут релиз
  },
  // ... другие helmreleases
]

```

### Конфигурация

* **URL** - URL Kubernetes API в формате `https://api.example.com`.

<a id="helmreleasesheaders"></a>
**Headers:**

|Название     |Обязательность|Описание                                                                |Пример        |
|-------------|--------------|------------------------------------------------------------------------|--------------|
|Authorization|**обязательно**|Токен для авторизации в Kubernetes API. [Подробнее](#helmreleasesauth)|Bearer <токен>|

**Параметры:**

Настраиваемые параметры отсутствуют.

## KafkaAcls

Источник данных со встроенным backend **KafkaAcls** собирает информацию о достпуных acls.

### Аутентификация

Платформа поддерживает аутентификацию в Kafka с помощью SASL/PLAIN и SASL/SCRAM. Документация: [Kafka SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain), [Kafka SASL/SCRAM](https://kafka.apache.org/documentation/#security_sasl_scram).

### Спецификация ответа

Платформа запрашивает информацию о настроенных ACL в Kafka. Полученные данные предоставляются в следующем формате:

```json
[
  {
    "Cluster": "string",             // Название кластера Kafka
    "ResourceType": "string",        // Тип ресурса (TOPIC, GROUP и т.д.)
    "ResourceName": "string",        // Имя ресурса
    "PatternType": "string",         // Тип паттерна (LITERAL, PREFIXED и т.д.)
    "Principal": "string",           // Principal пользователя (например: "User:Alice")
    "Host": "string",                // Хост (обычно "*" для любого хоста)
    "Operation": "string",           // Операция (READ, WRITE, DESCRIBE и т.д.)
    "PermissionType": "string"       // Тип разрешения (ALLOW, DENY)
  },
  // ... другие записи ACL
]
```

### Конфигурация

* **URL** - URL Kafka в формате `example.com`.

**Headers:**

Настраиваемые заголовки отсутствуют.

**Параметры:**

|Название         |Обязательность   |Описание                                                                                                                                                                                                    |Возможные значения                   |
|-----------------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------|
|SecurityProtocol | **обязательно** | Протокол для подключения к Kafka. [Подробнее](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
|SaslMechanism    | опционально     | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. [Подробнее](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
|User             | **обязательно** | Username пользователя для подключения к Kafka                                                                                                                                                              | -                                   |
|Pass             | **обязательно** | Password пользователя для подключения к Kafka                                                                                                                                                              | -                                   |

## KafkaBrokers

Источник данных со встроенным backend **KafkaBrokers** собирает информацию о доступных брокерах.

### Аутентификация

Платформа поддерживает аутентификацию в Kafka с помощью SASL/PLAIN и SASL/SCRAM. Документация: [Kafka SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain), [Kafka SASL/SCRAM](https://kafka.apache.org/documentation/#security_sasl_scram).

### Спецификация ответа

Платформа осуществляет несколько запросов к Kafka с целью получения сведений о доступных топиках. Полученные данные предоставляются в следующем формате:

```json
{
  "Cluster": "string",          // Название кластера Kafka (из kafkaBrokerMetadata.Cluster)
  "Leader": "boolean",          // Является ли брокер лидером-контроллером (true/false)
  "NodeID": "number",           // Уникальный ID брокера (из broker.NodeID)
  "Port": "number",             // Порт брокера (из broker.Port)
  "Host": "string",             // Хост брокера (из broker.Host)
  "Rack": "string|null",        // Рек (зона доступности) брокера (из broker.Rack)
  "Configs": {                  // Конфигурации брокера
    {
      "Name": "number",         // Уникальный ID брокера
      "Configs": [              // Массив отдельных конфигураций
        {
          "Key": "listeners",       // Ключ конфигурации
          "Value": "CLIENT://:9092,INTERNAL://:9094", // Значение конфигурации
          "Source": "STATIC_BROKER_CONFIG" // Источник конфигурации
        },
        {
          "Key": "log.retention.hours", // Ключ конфигурации
          "Value": 168,                 // Значение конфигурации
          "Source": "DEFAULT_CONFIG"    // Источник конфигурации
        },
        {
          "...": "..."                  // Другие параметры
          // Полный список параметров см. в официальной документации:
          // https://kafka.apache.org/documentation/#brokerconfigs
        }
      ]
    }
  }
}
```

### Конфигурация

* **URL** - URL Kafka в формате `example.com`.

**Headers:**

Настраиваемые заголовки отсутствуют.

**Параметры:**

|Название         |Обязательность   |Описание                                                                                                                                                                                                    |Возможные значения                   |
|-----------------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------|
|SecurityProtocol | **обязательно** | Протокол для подключения к Kafka. [Подробнее](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
|SaslMechanism    | опционально     | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. [Подробнее](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
|User             | **обязательно** | Username пользователя для подключения к Kafka                                                                                                                                                              | -                                   |
|Pass             | **обязательно** | Password пользователя для подключения к Kafka                                                                                                                                                              | -                                   |

## KafkaTopics

Источник данных со встроенным backend **KafkaTopics** собирает информацию о доступных топиках в Kafka.

### Аутентификация

Платформа поддерживает аутентификацию в Kafka с помощью SASL/PLAIN и SASL/SCRAM. Документация: [Kafka SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain), [Kafka SASL/SCRAM](https://kafka.apache.org/documentation/#security_sasl_scram).

### Спецификация ответа

Платформа осуществляет несколько запросов к Kafka с целью получения сведений о доступных топиках. Полученные данные предоставляются в следующем формате:

```json
{
  "Cluster": "string",           // Название кластера Kafka
  "Topic": "string",             // Название топика
  "ID": "string",                // Уникальный идентификатор топика
  "IsInternal": "boolean",       // Является ли топик внутренним (Пример внутреннего топика: __consumer_offsets)
  "Partitions": "number",        // Количество партиций в топике
  "Configs": {                   // Конфигурации топика (динамические параметры)
    "retention.ms": 604800000,   // Пример параметра: время хранения сообщений
    "cleanup.policy": "delete",  // Пример параметра: политика очистки
    "...": "..."                 // Другие параметры
    // Полный список параметров см. в официальной документации:
    // https://kafka.apache.org/documentation/#topicconfigs
  }
}
```

### Конфигурация

* **URL** - URL Kafka в формате `example.com`

**Headers:**

Настраиваемые заголовки отсутствуют.

**Параметры:**

|Название         |Обязательность   |Описание                                                                                                                                                                                                    |Возможные значения                   |
|-----------------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------|
|SecurityProtocol | **обязательно** | Протокол для подключения к Kafka. [Подробнее](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
|SaslMechanism    | опционально     | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. [Подробнее](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
|User             | **обязательно** | Username пользователя для подключения к Kafka                                                                                                                                                              | -                                   |
|Pass             | **обязательно** | Password пользователя для подключения к Kafka                                                                                                                                                              | -                                   |

## KubernetesDeployments

Источник данных со встроенным backend **KubernetesDeployments** возвращает список всех Deployment в кластере Kubernetes.

<a id="kubernetesdeploymentsauth"></a>

### Аутентификация

Платформа производит аутентификацию внутри кластера Kubernetes с помощью токена. Этот токен передается через конфигурацию Headers ([подробнее](#kubernetesdeploymentsheaders)).

Подробнее про аутентификацию в Kubernetes описано в [официальной документации](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

### Спецификация ответа

Платформа возвращает все Deployment в кластере Kubernetes. Спецификация ресурса Deployment доступна в [официальной документации](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/#DeploymentSpec).

### Конфигурация

* **URL** - URL Kubernetes API в формате `https://api.example.com`.

<a id="kubernetesdeploymentsheaders"></a>
**Headers:**

|Название     |Обязательность|Описание                                         |Пример        |
|-------------|--------------|-------------------------------------------------|--------------|
|Authorization|**обязательно**|Токен для авторизации в Kubernetes API. [Подробнее](#kubernetesdeploymentsauth)|Bearer <токен>|

**Параметры:**

Настраиваемые параметры отсутствуют.

## KubernetesIngresses

Источник данных со встроенным backend **KubernetesIngresses** возвращает список всех Ingress в кластере Kubernetes.

<a id="kubernetesingressesauth"></a>

### Аутентификация

Платформа производит аутентификацию внутри кластера Kubernetes с помощью токена. Этот токен передается через конфигурацию Headers ([подробнее](#kubernetesingressesheaders)).

Подробнее про аутентификацию в Kubernetes описано в [официальной документации](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

### Спецификация ответа

Платформа возвращает все Ingress в кластере Kubernetes. Спецификация ресурса Ingress доступна в [официальной документации](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/ingress-v1/).

### Конфигурация

* **URL** - URL Kubernetes API в формате `https://api.example.com`.

<a id="kubernetesingressesheaders"></a>
**Headers:**

|Название     |Обязательность|Описание                                         |Пример        |
|-------------|--------------|-------------------------------------------------|--------------|
|Authorization|**обязательно**|Токен для авторизации в Kubernetes API. [Подробнее](#kubernetesingressesauth)|Bearer <токен>|

**Параметры:**

Настраиваемые параметры отсутствуют.

## KubernetesNamespaces

Источник данных со встроенным backend **KubernetesNamespaces** возвращает список всех Namespace в кластере Kubernetes.

<a id="kubernetesnamespacesauth"></a>

### Аутентификация

Платформа производит аутентификацию внутри кластера Kubernetes с помощью токена. Этот токен передается через конфигурацию Headers ([подробнее](#kubernetesnamespacesheaders)).

Подробнее про аутентификацию в Kubernetes описано в [официальной документации](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

### Спецификация ответа

Платформа возвращает все Namespace в кластере Kubernetes. Спецификация ресурса Namespace доступна в [официальной документации](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/namespace-v1/).

### Конфигурация

* **URL** - URL Kubernetes API в формате `https://api.example.com`.

<a id="kubernetesnamespacesheaders"></a>
**Headers:**

|Название     |Обязательность|Описание                                         |Пример        |
|-------------|--------------|-------------------------------------------------|--------------|
|Authorization|**обязательно**|Токен для авторизации в Kubernetes API. [Подробнее](#kubernetesnamespacessauth)|Bearer <токен>|

**Параметры:**

Настраиваемые параметры отсутствуют.

## KubernetesResources

Источник данных со встроенным backend **KubernetesResources** возвращает список ресурсов Kubernetes. Тип возвращаемых ресурсов задается через параметры источника данных. Поддерживаются как встроенные в Kubernetes типы ресурсов, так и любые кастомные ресурсы.

<a id="kubernetesresourcesauth"></a>

### Аутентификация

Платформа производит аутентификацию внутри кластера Kubernetes с помощью токена. Этот токен передается через конфигурацию Headers ([подробнее](#kubernetesresourcesheaders)).

Подробнее про аутентификацию в Kubernetes описано в [официальной документации](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

### Спецификация ответа

Спецификация ответа зависит от типа ресурса, который планируется получить. Для получения точной информации необходимо обратиться к документации. Для встроенных в Kubernetes ресурсов доступна [официальная документация](https://kubernetes.io/docs/reference/kubernetes-api/).

В случае отсутствия возможности обратиться к документации, можно получить описание спецификации с помощью утилиты `kubectl`.

Пример для deployment:

`kubectl explain deployment --recursive`

Примерный вывод:

```yaml
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
    
FIELDS:
  apiVersion <string>
  kind <string>
  metadata <ObjectMeta>
    annotations <map[string]string>
    creationTimestamp <string>
...
```

Описание спецификации находится внутри `FIELDS:`.

### Конфигурация

* **URL** - URL Kubernetes API в формате `https://api.example.com`.

<a id="kubernetesresourcesheaders"></a>
**Headers:**

|Название     |Обязательность|Описание                                         |Пример        |
|-------------|--------------|-------------------------------------------------|--------------|
|Authorization|**обязательно**|Токен для авторизации в Kubernetes API. [Подробнее](#kubernetesresourcesauth)|Bearer <токен>|

**Параметры:**

|Название    |Обязательность |Описание                                                                                                                                                                                     |Возможные значения                       |
|------------|---------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
|apiGroup    |опционально    |API группа ресурса. Для ресурсов в core API группе поле не задается.                                                                                                                         |[См. определение требуемых Group и Version](#определение-требуемых-group-и-version)|
|version     |**обязательно**|Версия API ресурса.                                                                                                                                                                          |[См. определение требуемых Group и Version](#определение-требуемых-group-и-version)|
|isNamespaced|**обязательно**|Является ли ресурс namespaced или нет. Проверить, является ли ресурс namespaced можно с помощью команды `kubectl api-resources`                                                              |true, false                              |
|namespace   |опционально    |Namespace из которого будут собираться ресурсы. Если не указан, ресурсы будут собираться из всех namespace. Значение параметра учитывается только если значение **isNamespaced** равно `true`|                                         |
|resource    |**обязательно**|Название ресурса. Указывается маленькими буквами во множественном числе, как в поле NAME вывода команды `kubectl api-resources`.                                                             |                                         |

## Примеры конфигурации

Собрать все ingress ресурсы Kubernetes кластера:

```yaml
apiGroup: networking.k8s.io
version: v1
isNamespaced: true
resource: ingresses
```

Собрать все pod Kubernetes кластера:

```yaml
version: v1
isNamespaced: true
resource: pods
```

Собрать все pod Kubernetes кластера в namespace `d8-development-platform`:

```yaml
version: v1
isNamespaced: true
resource: pods
namespace: d8-development-platform
```

Собрать все namespace Kubernetes кластера:

```yaml
version: v1
isNamespaced: false
resource: namespaces
```

Собрать все кастомные ресурсы ModuleRelease Kubernetes кластера:

```yaml
apiGroup: deckhouse.io
version: v1alpha1
isNamespaced: false
resource: modulereleases
```

### Определение требуемых Group и Version

Каждому типу ресурса соответствует своя версия и группа. Полный список API-ресурсов с их группами и версиями: [документация Kubernetes](https://kubernetes.io/docs/reference/kubernetes-api/)

Если неизвестно, какие требуются Group и Version, можно попробовать подставить актуальные значения. Есть несколько вариантов как их посмотреть:

#### С помощью утилиты kubectl

Команда `kubectl explain` показывает `version` и `apiGroup` для ресурса.

Пример:

```bash
kubectl explain deployment
```

Вывод:

```yaml
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
    
FIELDS:
...
```

## NexusArtifacts

Источник данных со встроенным backend **NexusArtifacts** возвращает список артефактов в Nexus.

<a id="nexusartifactssauth"></a>

### Аутентификация

Аутентификация в Nexus осуществляется с помощью Basic Authentication. Для настройки аутентификации необходимо:

1. Сформировать строку username:password
2. Закодировать строку в Base64. Пример команды: `echo "username:password" | base64`
3. В конфигурации источника данных добавить заголовок `Authorization`. [Подробнее](#nexusartifactsheaders).

### Спецификация ответа

Платформа формирует список доступных репозиториев, затем по каждому репозиторию выполняется GET-запрос к API Nexus: `/service/rest/v1/components`. [Спецификация ответа](https://help.sonatype.com/en/components-api.html).

### Конфигурация

* **URL** - URL Nexus в формате `https://example.com`

<a id="nexusartifactsheaders"></a>
**Headers**

|Название      |Обязательность |Описание                                                         |Пример         |
|--------------|---------------|-----------------------------------------------------------------|---------------|
|Authorization |опционально    | Токен для авторизации в Nexus. [Подробнее](#nexusartifactsauth) | Basic <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## NexusRepositories

Источник данных со встроенным backend **NexusRepositories** возвращает список репозиториев в Nexus.

<a id="nexusrepositoriesauth"></a>

### Аутентификация

Аутентификация в Nexus осуществляется с помощью Basic Authentication. Для настройки аутентификации необходимо:

1. Сформировать строку username:password
2. Закодировать строку в Base64. Пример команды: `echo "username:password" | base64`
3. В конфигурации источника данных добавить заголовок `Authorization`. [Подробнее](#nexusrepositoriesheaders).

### Спецификация ответа

Платформа выполняет GET-запрос к API Nexus: `/service/rest/v1/repositories`. [Спецификация ответа](https://help.sonatype.com/en/repositories-api.html).

### Конфигурация

* **URL** - URL Nexus в формате `https://example.com`

<a id="nexusrepositoriesheaders"></a>
**Headers**

|Название      |Обязательность |Описание                                                           |Пример         |
|--------------|---------------|-------------------------------------------------------------------|---------------|
|Authorization |опционально    | Токен для авторизации в Nexus. [Подробнее](#nexusrepositoriesauth)| Basic <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## PrometheusMetrics

Источник данных со встроенным backend **PrometheusMetrics** возвращает результат запроса PromQL в Prometheus.

### Аутентификация

Аутентификация в prometheus зависит от настроек сервера. Платформа поддерживает аутентификацию, которая возможна путем настройки Headers ([подробнее](#prometheusmetricsheaders)).

Подробнее про способы настройки аутентификации в prometheus в [официальной документации](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#http_config).

**Пример Basic Authentication:**

1. Сформировать строку username:password
2. Закодировать строку в Base64. Пример команды: `echo "username:password" | base64`
3. В конфигурации источника данных добавить заголовок `Authorization`. Пример значения: `Basic <токен>`.

**Пример Bearer Token Auth:**

В конфигурации источника данных добавить заголовок `Authorization`. Пример значения: `Bearer <token>`.

### Спецификация ответа

Платформа выполняет GET-запрос к API Prometheus: `/api/v1/query`. [Спецификация ответа](https://prometheus.io/docs/prometheus/latest/querying/api/#instant-queries).

### Конфигурация

* **URL** - URL prometheus API в формате `https://example.com/api/v1/query`
* **Method** - GET
* **Тип query** - Prometheus
* **Query** - запрос в формате PromQL ([подробнее](https://prometheus.io/docs/prometheus/latest/querying/basics/)), на основе которого будет сформирован ответ.

**Headers:**

|Название      | Обязательность | Описание                                                                 |
|--------------|----------------|--------------------------------------------------------------------------|
|Authorization | опционально    | Токен для авторизации в Prometheus. [Подробнее](#prometheusmetricsauth). |

**Параметры:**

Настраиваемые параметры отсутствуют.

## SonarqubeProjects

Источник данных со встроенным backend **SonarqubeProjects** возвращает список проектов в SonarQube.

<a id="sonarqubeprojectsauth"></a>

### Аутентификация

Для работы источника данных необходим Sonarqube токен с типом User Token. Этот токен передается через конфигурацию Headers ([подробнее](#sonarqubeprojectsheaders)).

Подробнее о том, как получить Sonarqube токен можно узнать в [официальной документации](https://docs.sonarsource.com/sonarqube-server/latest/user-guide/managing-tokens/#generating-a-token).

### Спецификация ответа

Платформа выполняет GET-запрос к API SonarQube: `/api/projects/search`. Система собирает все доступные `components` и возвращает их в виде массива. [Спецификация `components`](https://next.sonarqube.com/sonarqube/web_api/api/projects/search).

### Конфигурация

* **URL** - URL Sonarqube в формате `https://example.com`

<a id="sonarqubeprojectsheaders"></a>
**Headers:**

|Название      |Обязательность |Описание                                                               |Пример         |
|--------------|---------------|-----------------------------------------------------------------------|---------------|
|Authorization |опционально    | Токен для авторизации в Sonarqube. [Подробнее](#sonarqubeprojectsauth)| Basic <токен> |

**Параметры:**

Настраиваемые параметры отсутствуют.

## GenericAPI

Источник данных со встроенным backend **GenericAPI** позволяет подключаться к любому REST API и получать данные в формате JSON. Поддерживает различные типы пагинации и настраиваемые параметры запроса.

### Аутентификация

GenericAPI поддерживает любые типы аутентификации через настройку заголовков HTTP. Наиболее распространенные способы:

**Bearer Token:**

```sh
Authorization: Bearer <токен>
```

**Basic Authentication:**

```sh
Authorization: Basic <base64-encoded-credentials>
```

**API Key:**

```sh
X-API-Key: <ключ>
```

### Спецификация ответа

GenericAPI возвращает массив объектов JSON. Структура данных зависит от конкретного API, к которому происходит подключение.

Пример типичного ответа:

```json
[
  {
    "id": 1,
    "name": "Example Item",
    "description": "Description of the item",
    "created_at": "2023-01-01T00:00:00Z"
  },
  {
    "id": 2,
    "name": "Another Item",
    "description": "Another description",
    "created_at": "2023-01-02T00:00:00Z"
  }
]
```

### Конфигурация

* **URL** - базовый URL API в формате `https://api.example.com`
* **Method** - HTTP метод (GET, POST, PUT, PATCH, DELETE)
* **Query Type** - тип запроса (Generic)
* **Query** - дополнительные query параметры (например, для фильтрации или поиска)

**Headers:**

Любые HTTP заголовки, необходимые для работы с API. Настраиваются в зависимости от требований конкретного API.

**Параметры:**

| Название         | Обязательность | Описание                                                                         | Возможные значения                      | Примеры                        | По умолчанию |
|------------------|----------------|----------------------------------------------------------------------------------|-----------------------------------------|--------------------------------|--------------|
| paginationType    | опционально    | Тип пагинации для обработки больших объемов данных                              | none, offset, cursor, page, link_header | none                           | none         |
| path              | опционально    | Путь к endpoint API, который будет добавлен к базовому URL                      | Любая строка, начинающаяся с /          | /api/v1/users, /projects       | ""           |
| dataPath          | опционально    | JSONPath для извлечения массива данных из ответа API                            | Любая строка                            | data, results.items, .         | .            |
| pageParam         | опционально    | Название параметра для номера страницы (для offset и page пагинации)            | Любая строка                            | page, _page, pageNumber        | page         |
| limitParam        | опционально    | Название параметра для количества элементов на странице (для offset пагинации)  | Любая строка                            | limit, per_page,_limit, size  | limit        |
| sizeParam         | опционально    | Название параметра для размера страницы (для page пагинации)                    | Любая строка                            | size, pageSize, _size          | size         |
| cursorParam       | опционально    | Название параметра для курсора (для cursor пагинации)                           | Любая строка                            | cursor, after, next            | cursor       |
| pageSize          | опционально    | Количество элементов на странице для пагинации                                  | Положительное целое число               | 10, 20, 50, 100                | 100          |
| requestBody       | опционально    | Тело запроса для POST/PUT/PATCH методов                                         | Любая строка                            | {"query": "example"}           | ""           |

## Типы пагинации

**none** - без пагинации, получает все данные одним запросом

**offset** - пагинация по номеру страницы и количеству элементов:
- Использует параметры `pageParam` и `limitParam`
- Пример: `?page=1&limit=20`

**page** - пагинация по номеру страницы и размеру:
- Использует параметры `pageParam` и `sizeParam`
- Пример: `?page=1&size=20`

**cursor** - пагинация по курсору:
- Использует параметр `cursorParam`
- Пример: `?cursor=eyJpZCI6MTIzfQ==`

**link_header** - пагинация через заголовок Link:
- Использует заголовок `Link` в ответе для определения следующей страницы
- Стандарт RFC 5988

## Примеры конфигурации

**JSONPlaceholder API (без пагинации):**

```yaml
url: https://jsonplaceholder.typicode.com
path: /users
paginationType: none
dataPath: .
```

**GitLab API (offset пагинация):**

```yaml
url: https://gitlab.example.com
path: /api/v4/projects
paginationType: offset
pageParam: page
limitParam: per_page
pageSize: 20
headers:
  Authorization: Bearer <token>
```

**GitHub API (link header пагинация):**

```yaml
url: https://api.github.com
path: /repos/owner/repo/issues
paginationType: link_header
headers:
  Authorization: Bearer <token>
  Accept: application/vnd.github.v3+json
```

**API с вложенными данными:**

```yaml
url: https://api.example.com
path: /data
dataPath: response.data.items
paginationType: offset
pageParam: page
limitParam: limit
query: "filter=active&sort=name"
```

**POST запрос с телом:**

```yaml
url: https://api.example.com
path: /search
method: POST
requestBody: '{"query": "example", "filters": {"status": "active"}}'
query: "include=metadata&format=json"
headers:
  Content-Type: application/json
  Authorization: Bearer <token>
```
