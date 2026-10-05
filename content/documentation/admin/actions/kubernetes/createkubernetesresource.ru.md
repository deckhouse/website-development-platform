---
title: CreateKubernetesResource
weight: 10
---

{{< alert level="info" >}}
Для выполнения действий необходимо наличие токена сервисного аккаунта Kubernetes.
{{< /alert >}}

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

| Название                     | Обязательность   | Описание                                                                                             |
| ---------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------- |
| manifests                    | Да               | Манифесты Kubernetes, которые будут применены                                                        |

### Ответ

Действие возвращает объект, в котором каждому применённому манифесту соответствует один ключ. Значение ключа — созданный или обновлённый объект в том виде, в котором его вернул API Kubernetes.

Ключ имеет вид `<KIND>_<NAMESPACE>_<NAME>` в нижнем регистре, дефисы (`-`) заменяются на подчёркивания (`_`):

- `<KIND>` — тип (kind) объекта, например `deployment`;
- `<NAMESPACE>` — неймспейс объекта. Если у объекта, относящегося к неймспейсу, не задан `metadata.namespace`, действие создаёт его в неймспейсе `default`. Для объектов уровня кластера, например Namespace, эта часть пустая;
- `<NAME>` — имя объекта.

Для примера запроса выше ответ содержит ключи `namespace__example1` и `namespace__example2`.
Для Deployment `my-app` без `metadata.namespace` ключ — `deployment_default_my_app`, а шаблон `{{ .response.deployment_default_my_app.metadata.uid }}` возвращает UID объекта.
