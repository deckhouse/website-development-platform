---
title: Создание и публикация шаблона микросервиса
weight: 40
description: Пошаговое создание пакета шаблона Go-сервиса для DDP — манифест, параметры, файлы, проверка утилитой ddp-template, публикация в хранилище образов, синхронизация и создание микросервиса из шаблона.
---

Инструкция описывает создание шаблона HTTP-сервиса на Go и его публикацию в реестр шаблонов портала. В результате шаблон отображается в галерее раздела главной «Создать микросервис», а созданный из него репозиторий содержит код, CI и Helm-чарт.

## Перед началом работы

Для выполнения инструкции подготовьте:

- хранилище образов контейнеров с OCI Distribution Spec v2, например `registry.example.com`, и токен с правами на запись в репозиторий `ddp-templates/go-service`;
- утилиту `ddp-template`;
- подключение этого хранилища в реестре шаблонов портала с префиксом репозиториев `ddp-templates`. Порядок подключения описан в разделе [«Реестр шаблонов»](../templates/registry/#подключение-хранилища).

## Структура пакета

Создайте директорию пакета со следующими файлами:

```text
go-service/
  template.yaml
  values.yaml
  .templateignore
  .editorconfig
  .gitlab-ci.yml
  go.mod
  cmd/{{ .slug }}/main.go
  deploy/helm/Chart.yaml
  deploy/helm/values.yaml
  deploy/helm/templates/deployment.yaml
```

Директория `cmd/{{ .slug }}` получит имя по идентификатору микросервиса.

### Манифест

Манифест `template.yaml` задаёт идентификатор шаблона и правила обновления. CI и Helm-чарт принадлежат шаблону и перезаписываются при обновлении, а `.editorconfig` добавляется, только если его нет в репозитории:

```yaml
id: go-service
name: Go service
description: HTTP service on Go with GitLab CI and Helm chart
language: Go
tags:
  - http
  - helm
update:
  strategy: selective_additive
  overwrite:
    - .gitlab-ci.yml
    - deploy/**
  addIfMissing:
    - .editorconfig
```

Значение `id` совпадает с именем репозитория пакета в хранилище: `ddp-templates/go-service`.

### Параметры

Файл `values.yaml` описывает параметры шаблона. Идентификатор, название микросервиса и система заполняются из контекста запуска и не отображаются в форме; порт пользователь вводит на шаге «Параметры»:

```yaml
type: object
required:
  - slug
  - name
  - system
  - port
properties:
  slug:
    type: string
    title: Service identifier
    x-ddp-from: context.microservice.slug
  name:
    type: string
    title: Service name
    x-ddp-from: context.microservice.name
  system:
    type: string
    title: System
    x-ddp-from: context.system.slug
  port:
    type: integer
    title: HTTP port
    description: Port the service listens on
    default: 8080
```

### Файлы репозитория

Файлы репозитория используют параметры шаблона. Файл `go.mod`:

```go-html-template
module {{ .slug }}

go 1.23
```

Файл `cmd/{{ .slug }}/main.go`:

```go-html-template
package main

import (
    "log"
    "net/http"
)

func main() {
    // Service {{ .name }} of system {{ .system }}.
    log.Fatal(http.ListenAndServe(":{{ .port }}", nil))
}
```

Файл `deploy/helm/values.yaml`:

```go-html-template
name: {{ .slug }}
system: {{ .system }}
port: {{ .port }}
```

Файл `deploy/helm/templates/deployment.yaml` использует синтаксис Helm `{{ .Values }}`. Чтобы портал не обрабатывал его как шаблон, добавьте путь в `.templateignore`:

```text
# Helm templates are copied as is.
deploy/helm/templates/**
```

## Проверка пакета

Проверьте пакет утилитой `ddp-template`:

```shell
ddp-template lint ./go-service
```

Команда проверяет манифест, схему параметров, `.templateignore`, списки путей обновления и выполняет пробную отрисовку. Код завершения `0` означает, что ошибок нет.

Чтобы посмотреть результат отрисовки, создайте файл `test-values.yaml` с тестовыми значениями и отрисуйте пакет:

```yaml
slug: billing-api
name: Billing API
system: billing
port: 8080
```

```shell
ddp-template render ./go-service --values ./test-values.yaml --output ./render-out
```

В директории `./render-out` появятся файлы репозитория: `cmd/billing-api/main.go`, `go.mod` с `module billing-api` и неизменённые Helm-шаблоны.

## Публикация

Опубликуйте версию `1.0.0` в хранилище образов:

```shell
export DDP_TEMPLATE_REGISTRY_TOKEN=<REGISTRY_TOKEN>
ddp-template publish ./go-service --version 1.0.0 \
  --registry registry.example.com \
  --repository-prefix ddp-templates
```

Где `<REGISTRY_TOKEN>` — токен хранилища образов с правами на запись.

Команда выводит JSON с полями `templateId`, `version`, `ociRef` и `sha256`. Опубликованную версию перезаписать нельзя: для изменений публикуйте новую версию.

## Синхронизация

Портал загружает версию при следующей синхронизации реестра шаблонов. Чтобы загрузить версию сразу:

1. Перейдите в раздел «Администрирование» → «Интеграции» → «Шаблоны» → «Запуски синхронизации».
1. Нажмите «Синхронизировать» и дождитесь статуса «Успех».
1. На вкладке «Шаблоны» найдите `go-service` версии `1.0.0` и убедитесь, что у неё статус «доступна» и тег «последняя».

Если поле «Идентификаторы репозиториев» подключения заполнено, добавьте в него `go-service` перед синхронизацией.

Если у версии статус «некорректна», причина указана в колонке «Причина статуса». Исправьте пакет и опубликуйте версию `1.0.1`.

## Привязка процессов

Привяжите к шаблону процессы в правилах маппинга главной:

1. Перейдите в раздел «Администрирование» → «Справочники» → «Правила маппинга» и откройте набор по умолчанию.
1. На вкладке «Маппинг каталога» раскройте «Микросервисы» → «Действия по шаблонам» и нажмите «Добавить шаблон».
1. Выберите шаблон `go-service` и процессы действий:
   - «Создать микросервис» — «Создать микросервис из шаблона»;
   - «Развернуть микросервис на окружение» — «Развернуть микросервис на окружение»;
   - «Удалить микросервис с окружения» — «Удалить микросервис с окружения»;
   - «Обновить микросервис из шаблона» — «Обновить микросервис из шаблона».
1. Нажмите «Сохранить».

## Проверка результата

Создайте микросервис из шаблона:

1. Откройте главную → «Создать микросервис» и выберите карточку «Go service».
1. На шаге «Сервис» заполните название `Billing API`, выберите систему и пространство GitLab и нажмите «Далее».
1. На шаге «Параметры» проверьте, что отображается только поле «HTTP port», и нажмите «Создать микросервис».
1. В окне «Запустить процесс» нажмите «Запустить» и дождитесь завершения процесса.

Шаблон работает, если выполнены условия:

- в GitLab создан проект `billing-api` с файлами `cmd/billing-api/main.go` и `.ddp/template.lock.yaml`;
- в lock-файле указаны `templateId: go-service` и `templateVersion: 1.0.0`;
- в карточке микросервиса на главной поля «Шаблон» и «Версия» содержат `go-service` и `1.0.0`.

Выпуск следующей версии шаблона и обновление микросервисов описаны в примере [«Выпуск новой версии шаблона и обновление микросервисов»](template-release-update/).
