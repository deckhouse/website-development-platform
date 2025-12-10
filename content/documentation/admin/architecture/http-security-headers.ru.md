---
title: HTTP-заголовки безопасности
weight: 50
---

DDP поддерживает следующие заголовки HTTP для повышения безопасности взаимодействия:

1. **Content Security Policy**
   - Настраиваемая политика безопасности контента
   - Подробное описание см. в разделе [Content Security Policy](#content-security-policy)

2. **X Frame Options**
   - Защита от clickjacking атак
   - По умолчанию: `SAMEORIGIN`
   - Настраивается в конфигурации DDP в секции `security.headers.xFrameOptions`
   - Возможные значения: `DENY`, `SAMEORIGIN`

3. **X Content Type Options**
   - Предотвращение MIME-sniffing
   - Значение: `nosniff`

4. **X XSS Protection**
   - Защита от XSS атак (для старых браузеров)
   - Значение: `1; mode=block`

5. **Referrer Policy**
   - Контроль передачи информации о реферере
   - Значение: `strict-origin-when-cross-origin`

6. **Permissions Policy** (ранее Feature Policy)
   - Отключение неиспользуемых функций браузера
   - Значение: `geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), accelerometer=()`

7. **Strict Transport Security (HSTS)**
   - Принудительное использование HTTPS
   - Значение: `max-age=31536000; includeSubDomains; preload`

## Конфигурация

Все заголовки настраиваются в конфигурации DDP в секции `security.headers`:

```yaml
security:
  headers:
    # Включить/выключить все заголовки безопасности
    enabled: true
    csp:
      # Разрешить загрузку изображений из внешних источников
      allowExternalImages: false
      # Разрешить использование iframe виджетов
      allowIframe: false
      # Дополнительные источники, добавляемые к директивам CSP: img-src, connect-src, frame-src, frame-ancestors. Например: https://example.com
      additionalSources: ""
    # Значение заголовка X-Frame-Options (DENY или SAMEORIGIN)
    xFrameOptions: "SAMEORIGIN"
```

Опция `enabled` управляет включением или отключением всех заголовков безопасности. При значении `true` все заголовки добавляются к HTTP ответам, при `false` — заголовки не добавляются.

## Где применяются

- **Frontend**: Заголовки добавляются на уровне веб-сервера в компоненте DDP Frontend для раздачи статических файлов и HTML страниц
- **Backend API**: Заголовки добавляются через middleware в компоненте DDP Backend для всех API ответов

{{< alert level="info" >}}
Все заголовки безопасности включены по умолчанию (`enabled: true`). При необходимости их можно отключить или настроить в конфигурации DDP.
{{< /alert >}}

## Content Security Policy

Content Security Policy (CSP) — это механизм безопасности, который помогает предотвратить XSS атаки и другие инъекции кода, ограничивая источники, из которых могут загружаться ресурсы (скрипты, стили, изображения и т.д.).

### CSP по умолчанию

```sh
default-src 'self';
script-src 'self';
style-src 'self' 'unsafe-inline';
font-src 'self';
img-src 'self' data:;
connect-src 'self' wss: ws:;
frame-src 'self';
frame-ancestors 'self';
worker-src 'self' blob:;
object-src 'none';
base-uri 'self';
form-action 'self';
upgrade-insecure-requests
```

{{< alert level="info" >}}
Отключение или ослабление политик может потребоваться в следующих случаях:
- Использование iframe виджетов (требует настройки `allowIframe: true`)
- Добавление иконок из внешних источников для объектов DDP (требует настройки `allowExternalImages: true`)
{{< /alert >}}
