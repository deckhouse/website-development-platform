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

| Название                   | Обязательность | Описание                                                                                           |
|----------------------------|----------------|----------------------------------------------------------------------------------------------------|
| manifests                  | Да             | Манифесты Kubernetes, которые будут применены                                                      |
