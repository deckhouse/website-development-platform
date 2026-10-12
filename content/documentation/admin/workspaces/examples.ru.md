---
title: Примеры
description: "Настройка шаблона DDP: новый файл, управляемая конфигурация, установка Go, скрипт запуска, пользовательский параметр и отдельный неймспейс."
weight: 40
---

Примеры расширяют встроенный шаблон рабочего пространства. После изменений сохраните черновик, проверьте его и опубликуйте новую версию. Для существующего пространства примените версию через [обновление](lifecycle/#обновление-параметров-и-версии).

## Добавление файла в проект

Файл `config/workspace.env` появится в checkout при первоначальной подготовке. Если такой файл уже есть в репозитории, шаблон сохранит его содержимое.

В `main.tf` внутри `module "workspace"` замените пустой `files = []`:

```hcl
files = [
  {
    path       = "config/workspace.env"
    policy     = "create_if_missing"
    content    = "APP_MODE=development\nLOG_LEVEL=debug\n"
    executable = false
  }
]
```

Если в `files` уже есть элементы, добавьте новый элемент в существующий список. Не создавайте второе поле `files` в том же блоке.

Путь относительно проекта `/workspace/code`; итоговый файл — `/workspace/code/config/workspace.env`. Редактирование этого файла пользователем сохраняется при Stop и Restore. Шаблон не перезаписывает его при последующей доставке с `create_if_missing`.

Для длинного содержимого создайте в редакторе пакетный файл `files/workspace.env`:

```text
APP_MODE=development
LOG_LEVEL=debug
```

В элементе `files` замените `content` чтением из пакета:

```hcl
content = file("${path.module}/files/workspace.env")
```

Это фрагмент внутри элемента списка, заданного в корневом `main.tf`. OpenTofu читает файл пакета при планировании, а Deckhouse Development Portal (DDP) доставляет принятое содержимое в проект. Добавление `files/workspace.env` в дерево шаблона без декларации доставки не создаёт файл в checkout.

## Управляемый файл

Политика `managed` позволяет обновлять конфигурацию вместе с версией шаблона и обнаруживать изменения пользователя. Добавьте в `files`:

```hcl
{
  path       = "config/tooling.env"
  policy     = "managed"
  content    = "FORMAT_ON_SAVE=true\n"
  executable = false
}
```

Создайте пространство из этой версии, затем измените файл в IDE. В новом черновике поменяйте желаемое содержимое, опубликуйте версию и подготовьте план обновления остановленного пространства. DDP покажет конфликт и предложит «Сохранить мои изменения» или «Заменить содержимым шаблона».

Решение относится к конкретному плану и содержимому файла. Если файл изменится после проверки, подготовьте новый план. Удаление ранее управляемого файла также выполняется явно через действие «Удалить этот файл», когда оно предложено планом.

## Установка Go

Скрипт устанавливает Go `1.25.12` для Linux `amd64` в постоянный каталог. Исполняемый файл включается в общий PATH через `$HOME/bin`, поэтому доступен в IDE и заданиях OpenCode. Установка выполняется в фазе `create`.

Целевая среда должна иметь сетевой доступ к `go.dev` и `dl.google.com`, доверенные TLS-сертификаты и утилиты `wget`, `sha256sum` и `tar`. Контрольная сумма взята [из списка выпусков Go](https://go.dev/dl/#go1.25.12). При смене версии меняйте URL, контрольную сумму и имя постоянного каталога вместе.

Создайте в редакторе файл `scripts/install-go.sh`:

```bash
set -eu

sdk="$HOME/sdk/go1.25.12"
mkdir -p "$HOME/sdk" "$HOME/bin"

if [ ! -x "$sdk/bin/go" ]; then
  archive="$(mktemp "$TMPDIR/go-archive.XXXXXX")"
  staging="$(mktemp -d "$HOME/sdk/go-staging.XXXXXX")"
  trap 'rm -f "$archive"; rm -rf "$staging"' EXIT
  wget -T 60 -O "$archive" https://go.dev/dl/go1.25.12.linux-amd64.tar.gz
  printf '%s  %s\n' '234828b7a89e0e303d2556310ee549fbcf253d28de937bac3da13d6294262ac1' "$archive" | sha256sum -c -
  tar -xzf "$archive" -C "$staging" --strip-components=1
  "$staging/bin/go" version
  mv "$staging" "$sdk"
  rm -f "$archive"
  trap - EXIT
fi

ln -sfn "$sdk/bin/go" "$HOME/bin/go"
"$HOME/bin/go" version
```

Добавьте в корень пакета файл `scripts.tf`:

```hcl
resource "ddp_script" "install_go" {
  name            = "install_go"
  phase           = "create"
  content         = file("${path.module}/scripts/install-go.sh")
  timeout_seconds = 240
  required        = true
  dependencies    = []
}
```

Лимит скрипта платформы должен быть не меньше `240` секунд. Скрипт выполняется через shell из `content`, поэтому для пакетного файла не требуется отдельный запуск через бит «Исполняемый». После публикации и создания пространства дождитесь готовности. В терминале IDE проверьте:

```shell
command -v go
go version
```

Ожидаемый путь — `/workspace/state/user-home/bin/go`, версия — `go1.25.12 linux/amd64`. После Stop и Restore SDK сохраняется. Для применения этого скрипта к существующему пространству обновите версию и согласуйте подготовку, показанную в плане.

## Скрипт каждого запуска

Скрипт фазы `start` выполняется после первоначальной подготовки и после каждого Restore. Он подходит для создания временной структуры или проверки установленного инструмента.

К предыдущему примеру добавьте:

```hcl
resource "ddp_script" "check_go" {
  name            = "check_go"
  phase           = "start"
  content         = "go version"
  timeout_seconds = 30
  required        = true
  dependencies    = ["install_go"]
}
```

`dependencies` содержит имя скрипта `install_go`. На первом запуске установка выполняется раньше проверки. При Restore DDP учитывает сохранённый результат `create` и запускает проверку снова. Если проверка завершится с ошибкой, обязательный скрипт блокирует готовность.

## Пользовательский параметр

Параметр памяти позволяет пользователю выбрать размер памяти исполнителя при создании и менять его через план обновления. В корневой файл `parameters.tf` добавьте:

```hcl
data "ddp_parameter" "runtime_memory_mib" {
  name         = "runtime_memory_mib"
  label        = "Память исполнителя, MiB"
  description  = "Размер памяти контейнера OpenCode"
  type         = "number"
  required     = true
  mutable      = true
  default_json = jsonencode(1024)
  minimum      = "512"
  maximum      = "4096"
}
```

В существующем вызове `module "workspace"` замените `runtime_resources`:

```hcl
runtime_resources = {
  cpu    = "500m"
  memory = "${jsondecode(data.ddp_parameter.runtime_memory_mib.value_json)}Mi"
}
```

Новое поле появится в форме создания. Backend отклонит значение вне диапазона. Изменение существующего пространства выполняется при остановленной среде и применяется только после подтверждения плана.

## Инструкции OpenCode

Файл инструкций задаёт правила работы с репозиторием. Добавьте в `module.workspace.files`:

```hcl
{
  path       = "AGENTS.md"
  policy     = "create_if_missing"
  content    = "Проверяйте изменённые пакеты. Перед удалением данных запросите подтверждение. Сохраняйте пользовательские изменения.\n"
  executable = false
}
```

В существующий объект `runtime_profile.engine` добавьте:

```hcl
templateConfig = {
  instructions = ["AGENTS.md"]
}
```

Сохраните обязательные поля `engine`. OpenCode получит инструкцию при запуске. Если репозиторий уже содержит `AGENTS.md`, политика `create_if_missing` сохранит его. Для централизованно обновляемой инструкции выберите `managed` и учитывайте конфликты.

## Отдельный неймспейс

Режим `per_workspace` задаёт каждому пространству неймспейс `ddp-<SUFFIX>`. Внешнее подключение должно разрешать создание и удаление Namespace. Общий служебный неймспейс Jobs остаётся в «Настройках подключения».

В существующей декларации `ddp_template` измените:

```hcl
namespace_mode = "per_workspace"
```

Удалите `allowed_namespaces`. В корневой файл `namespace.tf` добавьте:

```hcl
data "ddp_workspace" "target" {}

resource "kubernetes_namespace_v1" "workspace" {
  metadata {
    name = data.ddp_workspace.target.namespace
    labels = {
      "ddp.deckhouse.io/workspace" = data.ddp_workspace.target.uuid
    }
  }
}

resource "ddp_resource_policy" "namespace" {
  address = "kubernetes_namespace_v1.workspace"
  class   = "protected"
}
```

В существующий вызов `module "workspace"` добавьте:

```hcl
depends_on = [kubernetes_namespace_v1.workspace]
```

Namespace создаётся независимо от `start_count` и сохраняется при Stop. Подключение и namespace формы запуска берутся из опубликованной декларации. При Delete DDP проверяет принадлежность и наличие посторонних объектов; чужие объекты блокируют удаление неймспейса. Для общего существующего неймспейса используйте `existing` и не объявляйте управляемый Namespace.
