---
title: AI-агент
---

{{< alert level="warning" >}}
Экспериментальный функционал
{{< /alert >}}

AI-агент — это интеллектуальный помощник, встроенный в Deckhouse Development Platform. Он отвечает на вопросы о платформе, анализирует данные каталога и выполняет различные задачи с использованием инструментов MCP (Model Context Protocol).
AI-агент использует настраиваемые AI-провайдеры для обработки запросов и может работать с различными языковыми моделями, включая OpenAI GPT, Ollama и любые модели, доступные через совместимый REST API.

## Настройка AI-провайдеров

### Типы провайдеров

Платформа поддерживает три типа AI-провайдеров:

1. **OpenAI** — для работы с моделями OpenAI (GPT-4, GPT-3.5 и др.).
1. **Ollama** — для работы с локальными моделями через Ollama.
1. **Custom** — для работы с любым кастомным REST API, совместимым с форматом запросов/ответов AI-агента.

### Учетные данные для провайдеров

Для безопасного хранения токенов и ключей доступа к AI-провайдерам используется система учетных данных. Учетные данные шифруются и хранятся в базе данных, а в заголовки запросов к AI-провайдерам передаются через механизм шаблонизации.

#### Добавление учетных данных

Для добавления новых учетных данных для AI-провайдеров:

1. В форме создания или редактирования провайдера нажмите кнопку **Управление учетными данными**.
1. В открывшемся диалоге нажмите **Добавить учетные данные**.
1. Введите пару ключ-значение:
   - **Ключ**: название ключа учетных данных (например, `api_key`, `openai_token`, `bearer_token`).
   - **Значение**: сам токен или ключ доступа.
1. Нажмите **Сохранить**.

**Важно:**
- Ключ существующих учетных данных нельзя изменить. Для изменения ключа удалите старые учетные данные и создайте новые.
- Значение существующих учетных данных можно обновить, введя новое значение.
- Учетные данные шифруются при сохранении и никогда не передаются в веб-интерфейс после сохранения.

#### Использование учетных данных в заголовках

Вместо прямого указания токена в заголовках провайдера используйте синтаксис шаблонизации:

```sh
Authorization: Bearer {{ .credentials.api_key }}
```

где `api_key` — это название ключа, которое вы указали при добавлении учетных данных.

**Примеры:**

Вместо:

```sh
Authorization: Bearer sk-1234567890abcdef
```

Используйте:

```sh
Authorization: Bearer {{ .credentials.openai_api_key }}
```

**Преимущества использования шаблонизации:**
- Токены не хранятся в открытом виде в базе данных.
- Учетные данные можно обновить без изменения конфигурации провайдера.
- Один набор учетных данных можно использовать в нескольких провайдерах.

**Как это работает:**
1. Вы добавляете учетные данные через профиль пользователя.
2. В заголовках провайдера указываете шаблон `{{ .credentials.название_ключа }}`.
3. При выполнении запроса система автоматически заменяет шаблон на фактическое значение учетных данных.

### Создание провайдера OpenAI (ChatGPT)

Для настройки провайдера OpenAI выполните следующие шаги:

1. Откройте профиль пользователя и перейдите на вкладку **AI-провайдеры**.
1. Нажмите кнопку **Добавить**.
1. Заполните форму:
   - **Название**: `ChatGPT` (или любое другое удобное название).
   - **Провайдер**: выберите `OpenAI`.
   - **Модель**: укажите название модели, например `gpt-4` или `gpt-3.5-turbo`.
   - **URL**: `https://api.openai.com/v1/chat/completions`.
   - **Метод**: `POST`.
   - **Заголовки**: добавьте заголовок авторизации, используя учетные данные:

     ```sh
     Authorization: Bearer {{ .credentials.openai_api_key }}
     ```

     где `openai_api_key` — название ключа учетных данных (см. раздел [Учетные данные для провайдеров](#учетные-данные-для-провайдеров)).

1. Нажмите **Сохранить**.

### Создание провайдера Ollama

Для настройки локального провайдера Ollama:

1. Убедитесь, что Ollama запущена на вашем сервере.
1. Создайте нового провайдера:
   - **Название**: `Ollama Local`.
   - **Провайдер**: выберите `Ollama`.
   - **Модель**: укажите название модели, например `llama2` или `mistral`.
   - **URL**: `http://localhost:11434/api/generate` (или адрес вашего сервера Ollama).
   - **Метод**: `POST`.

1. Нажмите **Сохранить**.

### Создание кастомного провайдера

Для настройки кастомного REST API провайдера:

1. Создайте нового провайдера и выберите тип **Custom**.
1. Заполните основные поля (URL, метод, заголовки авторизации).
1. Нажмите **Сохранить**.

#### Поле ответа (Response Field)

Поле **Поле ответа** определяет путь к элементу JSON-ответа API, в котором содержится текст ответа модели. Этот путь используется для извлечения ответа из JSON, возвращаемого API провайдера.

**Формат указания пути:**
- Используйте точечную нотацию для доступа к вложенным полям.
- Для массивов используйте индекс, начиная с 0.
- Поле необязательно — если не указано, система попытается найти ответ автоматически.

**Примеры:**

Для OpenAI-совместимых API:

```json
choices.0.message.content
```

Это означает: взять первый элемент массива `choices`, затем поле `message`, затем поле `content`.

Для других форматов возможны значения `response.text`, `content`, `answer`.

**Как определить правильный путь:**
1. Выполните тестовый запрос к вашему API.
1. Изучите структуру JSON ответа.
1. Найдите поле, содержащее текст ответа модели.
1. Укажите путь к этому полю в точечной нотации.

#### Шаблон тела запроса (Request Body Template)

Поле **Шаблон тела запроса** определяет структуру JSON-запроса, который будет отправляться к API провайдера. В шаблоне можно использовать переменные, которые будут автоматически заменены на актуальные значения.

**Доступные переменные:**
- `{{.prompt}}` — текст вопроса пользователя (будет автоматически экранирован для JSON).
- `{{.model}}` — название модели, указанное в настройках провайдера.

**Примеры шаблонов:**

Для OpenAI-совместимых API:

```json
{
  "model": "{{.model}}",
  "messages": [
    {
      "role": "user",
      "content": "{{.prompt}}"
    }
  ],
  "temperature": 0.7
}
```

For APIs with a different format:

```json
{
"messages": [
{
"content": "{{.prompt}}",
"role": "user"
}
],
"model": "{{.model}}",
"stream": false
}
```

For APIs with a minimal request structure:

```json
{
"query": "{{.prompt}}",
"model_name": "{{.model}}"
}
```

**Important:**
- The template must be valid JSON.
- The `{{.prompt}}` and `{{.model}}` variables will be automatically replaced when sending the request.
- The `{{.prompt}}` value is automatically escaped for safe insertion into JSON.
- You can add any additional fields required for your API (temperature, max_tokens, etc.).

**Example of a complete custom provider setup:**

1. **Name**: `My Custom API`.
1. **Provider**: `Custom`.
1. **Model**: `my-model-v1`.
1. **URL**: `https://api.example.com/v1/chat`.
1. **Method**: `POST`.
1. **Headers**:

```sh
Authorization: Bearer {{ .credentials.api_key }}
Content-Type: application/json
```

where `api_key` is the name of the credential key (see the [Credentials for Providers](#credentials-for-providers) section).

1. **Response Field**: `choices.0.message.content`. 1. **Request body template**:

```json
{
"model": "{{.model}}",
"messages": [
{
"role": "user",
"content": "{{.prompt}}"
}
],
"temperature": 0.7,
"max_tokens": 1000
}
```

## Working with the AI Agent

### Opening Chat

The AI Agent is accessible through the chat sidebar on the right side of the interface. To open the chat, click the chat button in the lower right corner of the screen.

### Selecting a Provider

At the top of the chat is a provider selector. Select the desired provider from the list of available ones. If you only have one provider configured, it will be selected automatically.

### Available Tools (MCP Tools)

The AI Agent uses MCP (Model Context Protocol) tools to work with the platform. A complete list with descriptions, parameters, and examples is provided in the [MCP server documentation](../user/mcp-server/).

The model can call multiple tools in a single request if needed.

Key features:
- Retrieving and analyzing data from the platform catalog.
- MCP proxy to external services (GitLab, SonarQube, Kubernetes, etc.) via `get_external_data` — requests are executed with user credentials.

### Viewing Available Tools

The chat displays a list of all available tools with their descriptions and usage examples. You can:

- Expand the **Available Tools** section to view the list.
- Expand each tool to view its parameters and examples.
- Click on an example to paste it into the input field and immediately execute or edit the request.

### Features

1. **Data Analysis**: The AI agent doesn't just return raw data; it analyzes it and provides structured responses.
1. **Filtering**: You can request data with conditions, and the agent will perform the filtering.
1. **Aggregation**: The agent can count, group data, and provide statistics.

### Debugging

If the agent's response seems incorrect, you can view the debug logs by clicking the question mark icon next to the response. This will help you understand which tools were called and what data was retrieved.

## Usage Examples

### Example 1: Service Analysis

**Question:**

```sh
Get all services and show their name and creation date
```

**Result:**
The agent uses MCP tools to retrieve service data and returns a list with their name and creation date.

### Example 2: Action List

**Question:**

```sh
Get a list of actions
```

**Result:**
The agent will call the MCP tool and return a list of available platform actions.

## Recommendations

1. **Use specific questions**: The more specific the question, the more accurate the answer.
1. **Specify parameters explicitly**: If you know the name of a resource or parameter, specify it in the question.
1. **Experiment**: The AI agent understands natural language, so try different question formulations.