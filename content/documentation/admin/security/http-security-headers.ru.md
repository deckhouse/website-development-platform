---
title: HTTP-заголовки безопасности
weight: 50
---

Deckhouse Development Portal (DDP) поддерживает следующие HTTP-заголовки для повышения безопасности взаимодействия:

1. `Content-Security-Policy`
   - Настраиваемая политика безопасности контента.
   - Подробное описание приведено в разделе [Content Security Policy](#content-security-policy).

1. `X-Frame-Options`
   - Защита от clickjacking-атак.
   - Значение по умолчанию: `SAMEORIGIN`.
   - Настраивается в параметре модуля `security.headers.xFrameOptions`.
   - Возможные значения: `DENY`, `SAMEORIGIN`.

1. `X-Content-Type-Options`
   - Предотвращение MIME-sniffing.
   - Значение: `nosniff`.

1. `X-XSS-Protection`
   - Защита от XSS-атак (для устаревших браузеров).
   - Значение: `1; mode=block`.

1. `Referrer-Policy`
   - Контроль передачи информации о реферере.
   - Значение: `strict-origin-when-cross-origin`.

1. `Permissions-Policy` (ранее Feature-Policy)
   - Ограничение использования функций браузера.
   - Значение в ответах DDP Frontend: `geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), accelerometer=()`.
   - Значение в ответах DDP Backend: `geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), accelerometer=(), autoplay=(), encrypted-media=()`.

1. `Strict-Transport-Security` (HSTS)
   - Принудительное использование HTTPS.
   - Значение: `max-age=31536000; includeSubDomains; preload`.
   - DDP Backend добавляет заголовок, только если TLS-соединение завершается на самом DDP Backend. DDP Frontend этот заголовок не добавляет.

{{< alert level="info" >}}
При стандартной установке TLS-соединение завершается на Ingress-контроллере, поэтому DDP не добавляет заголовок `Strict-Transport-Security`. Чтобы включить HSTS, настройте его на Ingress-контроллере, например в параметре [`hsts`](/modules/ingress-nginx/cr.html#ingressnginxcontroller-v1-spec-hsts) IngressNginxController.
{{< /alert >}}

## Конфигурация

Заголовки настраиваются в параметре `security.headers` модуля `development-platform`: в ModuleConfig `development-platform` (`spec.settings.security.headers`) или в настройках модуля в веб-интерфейсе Deckhouse Platform.

Пример фрагмента ModuleConfig со значениями по умолчанию:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  settings:
    security:
      headers:
        # Включить или выключить все заголовки безопасности.
        enabled: true
        csp:
          # Разрешить загрузку изображений из любых источников.
          allowExternalImages: false
          # Разрешить встраивание портала в iframe на любых сайтах и загрузку любых сайтов в iframe портала.
          allowIframe: false
          # Дополнительные источники через пробел, добавляемые к директивам CSP img-src, connect-src, frame-src, frame-ancestors.
          # Например: https://example.com.
          additionalSources: ""
        # Значение заголовка X-Frame-Options (DENY или SAMEORIGIN).
        xFrameOptions: "SAMEORIGIN"
```

Параметр `enabled` включает или отключает все заголовки безопасности. При значении `true` заголовки добавляются к HTTP-ответам, при значении `false` заголовки не добавляются.

## Применение

- DDP Frontend: заголовки добавляются на уровне веб-сервера для раздачи статических файлов и HTML-страниц.
- DDP Backend: заголовки добавляются через middleware для всех ответов API.

{{< alert level="info" >}}
Все заголовки безопасности включены по умолчанию (`enabled: true`). При необходимости их можно отключить или настроить в параметре модуля `security.headers`.
{{< /alert >}}

## Content Security Policy

Content Security Policy (CSP) — это механизм безопасности, который помогает предотвратить XSS-атаки и другие инъекции кода, ограничивая источники, из которых могут загружаться ресурсы (скрипты, стили, изображения и т. д.).

### CSP по умолчанию

DDP Frontend и DDP Backend отправляют разные политики CSP.

Политика DDP Frontend по умолчанию:

```text
default-src 'self';
script-src 'self';
style-src 'self' 'unsafe-inline';
font-src 'self' data:;
img-src 'self' data:;
connect-src 'self' wss: ws:;
frame-src 'self';
worker-src 'self' blob:;
object-src 'none';
base-uri 'self';
form-action 'self';
frame-ancestors 'self';
upgrade-insecure-requests;
```

Политика DDP Backend по умолчанию:

```text
default-src 'self';
script-src 'none';
style-src 'none';
img-src 'none';
font-src 'none';
connect-src 'self';
frame-src 'none';
frame-ancestors 'none';
object-src 'none';
base-uri 'self';
form-action 'self';
upgrade-insecure-requests
```

### Изменение CSP

Параметры `security.headers.csp` изменяют политику следующим образом:

- `allowExternalImages: true` — разрешает изображения из любых источников: в политике DDP Frontend директива `img-src` принимает значение `'self' data: *`, в политике DDP Backend — `*`. Используется, например, для иконок объектов DDP из внешних источников.
- `allowIframe: true` — задаёт значение `*` для директив `frame-src` и `frame-ancestors` в обеих политиках. Используется, например, для iframe-виджетов.
- `additionalSources` — добавляет перечисленные источники к директивам `img-src`, `connect-src`, `frame-src` и `frame-ancestors` в обеих политиках.

{{< alert level="warning" >}}
При `allowIframe: true` портал можно встроить в iframe на любом сайте, а в iframe-виджетах портала можно открыть любой сайт. Заголовок `X-Frame-Options` при этом по-прежнему отправляется, но современные браузеры при наличии директивы `frame-ancestors` его не учитывают. Это снижает защиту от clickjacking-атак.

Чтобы разрешить только доверенные сайты, оставьте `allowIframe: false` и перечислите их в параметре `additionalSources`.
{{< /alert >}}
