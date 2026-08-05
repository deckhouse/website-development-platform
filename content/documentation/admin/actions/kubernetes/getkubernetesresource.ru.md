---
title: GetKubernetesResource
weight: 20
---


{{< alert level="info" >}}
Для выполнения действий необходимо наличие токена сервисного аккаунта Kubernetes.
{{< /alert >}}

GetKubernetesResource — получает ресурс из кластера Kubernetes.

### Пример запроса

```yaml
group: managed-services.deckhouse.io
version: v1alpha1
resource_type: postgres
resource_name: example-postgres
namespace: default
```

### Спецификация запроса

| Название                    | Обязательность   | Описание                                                                           | Возможные значения                                                              |
| --------------------------- | ---------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| group                       | Да               | API-группа ресурса. Указывает, к какой группе API относится запрашиваемый объект   | [Определение требуемых Group и Version](#определение-требуемых-group-и-version) |
| version                     | Да               | Версия API ресурса                                                                 | [Определение требуемых Group и Version](#определение-требуемых-group-и-version) |
| resource_type               | Да               | Тип получаемого ресурса                                                            | -                                                                               |
| resource_name               | Да               | Название конкретного ресурса, который необходимо получить                          | -                                                                               |
| namespace                   | Да               | Неймспейс, в котором находится ресурс                                              | -                                                                               |

### Ответ

При успешном выполнении действие возвращает объект ресурса в поле `resource`. Если ресурс не найден, действие завершается с ошибкой.

| Название     | Описание                                        |
| ------------ | ----------------------------------------------- |
| `resource`   | Объект ресурса в формате Kubernetes             |

### Определение требуемых Group и Version

Каждому типу ресурса соответствует своя группа API (Group) и версия (Version).
Полный список API-ресурсов с их группами и версиями приведён [в документации Kubernetes](https://kubernetes.io/docs/reference/kubernetes-api/).

Если неизвестно, какие требуются группы API и версии, можно использовать актуальные значения.
Существует несколько способов их определить:

#### С помощью утилиты `d8 k`

Команда `d8 k explain` показывает `apiVersion` для ресурса.

Пример:

```bash
d8 k explain deployment
```

Пример вывода:

```yaml
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
    
FIELDS:
...
```

#### С помощью документации

1. Найдите нужный ресурс (например, Deployment).
1. В заголовке указана API Group и версия. Пример для Deployment:

   ```yaml
   apiVersion: apps/v1
   ```

   Здесь:
    * «API Group» — apps;
    * «Version» — v1.
