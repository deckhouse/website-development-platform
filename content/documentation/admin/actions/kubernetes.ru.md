# Kubernetes

## Содержание

- CreateKubernetesResource
- GetKubernetesResource
- DeleteKubernetesResource

## CreateKubernetesResource

CreateKubernetesResource — создаёт новый ресурс или ресурсы в кластере Kubernetes или обновляет существующие.

### Пример запроса

```yaml
manifests:
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: example1
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: example2
```

### Спецификация запроса

| Название                   | Обязательность | Описание                                                                                           |
|----------------------------|----------------|----------------------------------------------------------------------------------------------------|
| manifests                  | Да             | Манифесты Kubernetes, которые будут применены                                                      |

### Учётные данные

* `token` — токен сервисного аккаунта в Kubernetes.

## GetKubernetesResource

GetKubernetesResource — получает ресурс из Kubernetes кластера.

### Пример запроса

```yaml
group: managed-services.deckhouse.io
version: v1alpha1
resource_type: postgres
resource_name: example-postgres
namespace: default
```

### Спецификация запроса

| Название                  | Обязательность | Описание                                                                     | Возможные значения                                                                                                                                                                                                                                                                                                                                                                                                                      |
|---------------------------|----------------|------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| group                     | Да             | API-группа ресурса. Указывает, к какой группе API относится запрашиваемый объект | [Определение требуемых Group и Version](#определение-требуемых-group-и-version)                                                                                                                                                                                                                                                                                                                                                         |
| version                   | Да             | Версия API ресурса                                                           | [Определение требуемых Group и Version](#определение-требуемых-group-и-version)                                                                                                                                                                                                                                                                                                                                                         |
| resource_type             | Да             | Название ресурса во множественном числе, как в поле NAME вывода команды `kubectl api-resources` | - |
| resource_name             | Да             | Название конкретного ресурса, который необходимо получить                    | -                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| namespace                 | Да             | Неймспейс, в котором находится ресурс                                        | -                                                                                                                                                                                                                                                                                                                                                                                                                                       |

### Ответ

При успешном выполнении действие возвращает объект ресурса в поле `resource`. Если ресурс не найден, действие завершается с ошибкой.

| Название   | Описание                                      |
|------------|-----------------------------------------------|
| `resource` | Объект ресурса в формате Kubernetes           |

### Учётные данные

* `token` — токен сервисного аккаунта в Kubernetes.

## DeleteKubernetesResource

DeleteKubernetesResource — удаляет существующий ресурс в кластере Kubernetes.

### Пример запроса

```yaml
group: apps
version: v1
resource_type: deployments
resource_name: nginx-deployment
namespace: example
```

### Спецификация запроса

| Название                  | Обязательность  | Описание                                                                     | Возможные значения                                                                                                                                                                                                                                                                                                                                                                                                                      |
|---------------------------|-----------------|------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| group                     | Да              | API-группа ресурса. Указывает, к какой группе API относится удаляемый объект | [Определение требуемых Group и Version](#определение-требуемых-group-и-version)                                                                                                                                                                                                                                                                                                                                                         |
| version                   | Да | Версия API ресурса                                                           | [Определение требуемых Group и Version](#определение-требуемых-group-и-version)                                                                                                                                                                                                                                                                                                                                                         |
| resource_type             | Да | Тип удаляемого ресурса                                                       | pods, services, deployments, statefulsets, daemonsets, replicasets, jobs, cronjobs, nodes, namespaces, configmaps, secrets, persistentvolumes, persistentvolumeclaims, limitranges, resourcequotas, horizontalpodautoscalers, ingresses, networkpolicies, serviceaccounts, roles, clusterroles, rolebindings, clusterrolebindings, podsecuritypolicies, storageclasses, volumeattachments, events, endpoints, customresourcedefinitions |
| resource_name             | Да | Название конкретного ресурса, который необходимо удалить                     | -                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| namespace                 | Да | Неймспейс, в котором находится ресурс                                        | -                                                                                                                                                                                                                                                                                                                                                                                                                                       |

### Учётные данные

* `token` — токен сервисного аккаунта в Kubernetes.

### Определение требуемых Group и Version

Каждому типу ресурса соответствует своя группа API (Group) и версия (Version).
Полный список API-ресурсов с их группами и версиями приведён [в документации Kubernetes](https://kubernetes.io/docs/reference/kubernetes-api/).

Если неизвестно, какие требуются Group и Version, можно использовать актуальные значения.
Существует несколько способов их определить:

#### С помощью утилиты `d8 k`

Команда `d8 k explain` показывает `apiVersion` для ресурса.

Пример:

```bash
d8 k explain deployment
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

#### С помощью документации

Как искать в документации:

1. Найдите нужный ресурс (например, Deployment).
1. В заголовке будет указана API Group и версия. Пример для Deployment:

   ```yaml
   apiVersion: apps/v1
   ```

   Здесь:
    * «API Group» — apps;
    * «Version» — v1.