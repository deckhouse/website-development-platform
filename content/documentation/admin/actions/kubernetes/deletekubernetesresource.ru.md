---
title: DeleteKubernetesResource
weight: 30
---

{{< alert level="info" >}}
Для выполнения действий необходимо наличие токена сервисного аккаунта Kubernetes.
{{< /alert >}}

DeleteKubernetesResource — удаляет существующий ресурс в кластере Kubernetes.

### Пример запроса

```yaml
group: apps
version: v1
resource_type: deployments
resource_name: nginx-deployment
namespace: example
ignore_not_found: true
```

### Спецификация запроса

| Название                    | Обязательность    | Описание                                                                       | Возможные значения                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --------------------------- | ----------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| group                       | Да                | API-группа ресурса. Указывает, к какой группе API относится удаляемый объект   | [Определение требуемых Group и Version](getkubernetesresource/#определение-требуемых-group-и-version)                                                                                                                                                                                                                                                                                                                                     |
| version                     | Да                | Версия API ресурса                                                             | [Определение требуемых Group и Version](getkubernetesresource/#определение-требуемых-group-и-version)                                                                                                                                                                                                                                                                                                                                     |
| resource_type               | Да                | Тип удаляемого ресурса                                                         | pods, services, deployments, statefulsets, daemonsets, replicasets, jobs, cronjobs, nodes, namespaces, configmaps, secrets, persistentvolumes, persistentvolumeclaims, limitranges, resourcequotas, horizontalpodautoscalers, ingresses, networkpolicies, serviceaccounts, roles, clusterroles, rolebindings, clusterrolebindings, podsecuritypolicies, storageclasses, volumeattachments, events, endpoints, customresourcedefinitions   |
| resource_name               | Да                | Название конкретного ресурса, который необходимо удалить                       | -                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| namespace                   | Да                | Неймспейс, в котором находится ресурс                                          | -                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ignore_not_found            | Нет               | При значении `true` удаление объекта, которого нет в кластере, завершается успешно. По умолчанию `false`: удаление отсутствующего объекта завершается ошибкой | `true`, `false` |

### Примечание

Параметр `ignore_not_found` не подавляет другие ошибки API Kubernetes. Используйте его в процессах удаления, которые могут запускаться повторно.
