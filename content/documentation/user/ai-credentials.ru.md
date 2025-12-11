---
title: Учетные данные AI-провайдеров
---

Система управления учетными данными для AI-провайдеров позволяет каждому пользователю хранить зашифрованные ключи для использования в заголовках HTTP-запросов к AI-провайдерам.

## Обзор

Учетные данные AI-провайдеров — это зашифрованная мапа ключ-значение, которая хранится для каждого пользователя отдельно. Эти данные используются для автоматической подстановки в заголовки HTTP-запросов к AI-провайдерам через плейсхолдеры.

### Основные возможности

- **Глобальная мапа credentials** — каждый пользователь имеет свою мапу учетных данных, не привязанную к конкретному провайдеру
- **Неограниченное количество credentials** — можно добавить любое количество ключ-значение пар
- **Автоматическое шифрование** — все значения автоматически шифруются при сохранении
- **Замена плейсхолдеров** — автоматическая замена плейсхолдеров в заголовках провайдеров
- **Интеграция в UI** — полная интеграция в интерфейс для управления credentials

## Использование через интерфейс

### Добавление учетных данных

1. Откройте профиль пользователя
2. Перейдите на вкладку **AI-провайдеры**
3. Нажмите **Управление учетными данными** на любом провайдере
4. Добавьте ключ-значение пары (например, `openai_api_key` → `sk-...`)
5. Сохраните изменения

### Использование credentials в заголовках провайдера

При настройке AI-провайдера в заголовках можно использовать плейсхолдеры для автоматической подстановки учетных данных:

**Пример 1: Использование API ключа OpenAI**

```
Header: Authorization
Value: Bearer {{credential:openai_api_key}}
```

**Пример 2: Использование API ключа Anthropic**

```
Header: X-API-Key
Value: {{credential:anthropic_api_key}}
```

**Пример 3: Кастомный ключ**

```
Header: Authorization
Value: Bearer {{credential:custom_api_key}}
```

При выполнении запроса к AI-провайдеру плейсхолдеры автоматически заменяются на реальные значения из мапы credentials пользователя.

## Использование через API

### Базовый URL

Все endpoints доступны по базовому пути: `/api/v2/users/me/ai-credentials`

**Аутентификация**: Все endpoints требуют авторизации через API Token или Session.

### Получить все учетные данные

**GET** `/api/v2/users/me/ai-credentials`

Получить всю мапу credentials текущего пользователя.

**Пример запроса:**

```bash
curl -X GET "https://api.example.com/api/v2/users/me/ai-credentials" \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

**Пример ответа:**

```json
{
  "response": {
    "credentials": {
      "openai_api_key": "sk-...",
      "anthropic_api_key": "sk-ant-..."
    }
  }
}
```

### Установить все учетные данные

**PUT** `/api/v2/users/me/ai-credentials`

Установить всю мапу credentials (заменяет существующие).

**Пример запроса:**

```bash
curl -X PUT "https://api.example.com/api/v2/users/me/ai-credentials" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "credentials": {
      "openai_api_key": "sk-...",
      "anthropic_api_key": "sk-ant-...",
      "custom_api_key": "custom-value"
    }
  }'
```

### Установить или обновить одну пару ключ-значение

**PUT** `/api/v2/users/me/ai-credentials/:key`

Установить или обновить одну пару ключ-значение.

**Пример запроса:**

```bash
curl -X PUT "https://api.example.com/api/v2/users/me/ai-credentials/openai_api_key" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "value": "sk-your-openai-api-key"
  }'
```

### Удалить одну пару ключ-значение

**DELETE** `/api/v2/users/me/ai-credentials/:key`

Удалить одну пару ключ-значение по ключу.

**Пример запроса:**

```bash
curl -X DELETE "https://api.example.com/api/v2/users/me/ai-credentials/openai_api_key" \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

### Удалить все учетные данные

**DELETE** `/api/v2/users/me/ai-credentials`

Удалить все credentials пользователя.

**Пример запроса:**

```bash
curl -X DELETE "https://api.example.com/api/v2/users/me/ai-credentials" \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

## Как работает замена плейсхолдеров

При вызове AI-агента через LLM Router:

1. AI-агент получает мапу credentials пользователя
2. Мапа передается в опции запроса
3. Функция замены плейсхолдеров ищет все вхождения `{{credential:key}}` в заголовках
4. Плейсхолдеры заменяются на реальные значения из мапы credentials
5. Заголовки с реальными значениями используются в HTTP-запросах к AI-провайдерам

**Пример:**

До замены:
```json
{
  "Headers": {
    "Authorization": ["Bearer {{credential:openai_api_key}}"]
  }
}
```

После замены:
```json
{
  "Headers": {
    "Authorization": ["Bearer sk-actual-key-value"]
  }
}
```

## Безопасность

### Шифрование

- Все значения credentials автоматически шифруются перед сохранением в базу данных с использованием AES-GCM
- Значения расшифровываются только при чтении и передаче в LLM router
- В API ответах чувствительные заголовки маскируются (показывается "MASKED")

### Изоляция пользователей

- Каждый пользователь видит только свои credentials
- Credentials одного пользователя недоступны другим пользователям
- Уникальное ограничение: один ключ на пользователя (`UNIQUE(user_uuid, key)`)

## Решение проблем

### Credentials не найдены

Если при использовании `{{credential:key}}` в заголовках вы получаете предупреждение "Credential not found":

1. Убедитесь, что credential с таким ключом добавлен в "Управление учетными данными"
2. Проверьте правильность написания ключа в плейсхолдере (регистр имеет значение)
3. Убедитесь, что вы используете credentials того же пользователя, который создал провайдер

### Credentials не отображаются на фронтенде

Если credentials не отображаются на фронтенде:

1. Проверьте, что API возвращает данные: `GET /api/v2/users/me/ai-credentials`
2. Проверьте консоль браузера на наличие ошибок
3. Убедитесь, что компонент управления credentials правильно получает данные

### Плейсхолдеры не заменяются

Если плейсхолдеры не заменяются в заголовках:

1. Проверьте синтаксис плейсхолдера: `{{credential:key}}` (без пробелов)
2. Убедитесь, что ключ существует в мапе credentials пользователя
3. Проверьте логи выполнения запроса для диагностики

## Примеры использования

### Пример 1: Настройка OpenAI провайдера с credentials

1. Добавьте credential через интерфейс:
   - Ключ: `openai_api_key`
   - Значение: `sk-your-actual-key`

2. Создайте или отредактируйте провайдера OpenAI:
   - **Заголовки**: `Authorization: Bearer {{credential:openai_api_key}}`

3. При использовании AI-агента плейсхолдер автоматически заменится на реальный ключ

### Пример 2: Использование нескольких credentials

1. Добавьте несколько credentials:
   - `openai_api_key` → `sk-...`
   - `anthropic_api_key` → `sk-ant-...`
   - `custom_header_value` → `custom-value`

2. Используйте их в разных провайдерах:
   - OpenAI провайдер: `Authorization: Bearer {{credential:openai_api_key}}`
   - Anthropic провайдер: `X-API-Key: {{credential:anthropic_api_key}}`
   - Custom провайдер: `Custom-Header: {{credential:custom_header_value}}`

### Пример 3: Программное управление через API

```bash
# Добавить credential
curl -X PUT "https://api.example.com/api/v2/users/me/ai-credentials/openai_api_key" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"value": "sk-..."}'

# Получить все credentials
curl -X GET "https://api.example.com/api/v2/users/me/ai-credentials" \
  -H "Authorization: Bearer YOUR_API_TOKEN"

# Удалить credential
curl -X DELETE "https://api.example.com/api/v2/users/me/ai-credentials/openai_api_key" \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

## Важные замечания

1. **Глобальная мапа**: Credentials не привязаны к конкретному провайдеру — одна мапа для всех провайдеров пользователя
2. **Уникальность ключей**: Один ключ на пользователя — при добавлении ключа с существующим именем значение будет обновлено
3. **Автоматическая замена**: Плейсхолдеры заменяются только в заголовках провайдеров, не в теле запроса
4. **Безопасность**: Никогда не храните credentials в открытом виде в настройках провайдера — используйте плейсхолдеры

