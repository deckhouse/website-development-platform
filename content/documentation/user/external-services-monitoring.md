---
title: External services health monitoring
menuTitle: External services monitoring
d8Edition: ee
moduleStatus: experimental
---

Full description of capabilities and instructions for using health checks for external services.

## 1. Overview

External services health monitoring allows you to:

- **Periodically check availability** of an external service over HTTP(S).
- **Store status** (healthy / unhealthy / unknown) and **check history** (time, response code, response time, errors).
- **Notify** on status change (webhook or system alert).
- **Configure schedule** for checks using cron expressions (for example, every 5 minutes).

Checks run in the background on the backend. A separate schedule is created for each external service with monitoring enabled; when a service is updated or removed, the schedule is updated automatically.

---

## 2. Capabilities

### 2.1. Enabling and disabling monitoring

- **Enable Monitoring** — master switch for monitoring for the service.
- **Enable Health Check** — turn the checks themselves on or off within monitoring (with monitoring on, you can temporarily disable only the checks).

When monitoring is disabled, the schedule for that service is removed and no new checks run. Status and history remain in the database until the service is deleted.

### 2.2. Health check settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| **Schedule (cron)** | How often to run checks, in cron format (minute hour day month day_of_week). | `*/5 * * * *` (every 5 minutes) |
| **Timeout** | Timeout for a single HTTP request (e.g. `10s`, `30s`). | `10s` |
| **Method** | HTTP method for the request. | GET |
| **Path** | Path relative to the service base URL (e.g. `/health`, `/api/status`). | `/health` |
| **Expected Status Code** | HTTP response code that indicates a successful check. | 200 |
| **Request Body** | Request body in YAML/JSON (for POST/PUT/PATCH). | — |
| **Headers** | Additional request headers. | — |

Final check URL: `{service URL}{Path}` (e.g. `https://api.example.com/health`).

**Statuses:**

- **healthy** — request completed, response code matches expected.
- **unhealthy** — network error, timeout, response code does not match, or server returned 5xx/429.
- **unknown** — check has not run yet or data is missing.

### 2.3. Notifications

You can send a notification when status changes:

| Type | Description |
|------|-------------|
| **None** | Notifications disabled. |
| **Webhook** | HTTP request to the specified URL when status becomes unhealthy or when it recovers (healthy). URL, method, and headers are configurable; body can use templates. |
| **System Alert** | Create a system alert in the platform (if this feature is enabled). Alert name and description templates can be set. |

**Message templates** (for webhook and alerts):

- `{{.service.name}}` — external service name.
- `{{.service.url}}` — service URL.
- `{{.status}}` — new status (healthy / unhealthy).
- `{{.error}}` — error message (if any).
- `{{.responseTime}}` — response time in milliseconds.

Example text on failure:  
`Service {{.service.name}} is unhealthy. Error: {{.error}}`

### 2.4. Check history

- **Each** external service has its own check history.
- By default **no more than 10 recent entries** per service are kept; older ones are removed automatically after each new check.
- Each entry includes: check time, status, response time (ms), error message (if any).

The limit of 10 entries per service is defined in code and is not configurable.

---

## 3. How to use

### 3.1. Via the web UI

1. **Go to external services**  
   Administration → External Services.

2. **Create or open an external service**  
   - Create: **Connect** button, fill in name, slug, URL, and if needed system account and headers.  
   - Edit: pencil icon next to the row.

3. **Monitoring tab**  
   - Turn on **Enable Monitoring**.  
   - Turn on **Enable Health Check**.  
   - Set **Schedule (cron)** — e.g. `*/5 * * * *` for every 5 minutes.  
   - Optionally change **Timeout**, **Method**, **Path**, **Expected Status Code**, request body, and headers.  
   - In **Notifications**, choose notification type (None / Webhook / System Alert) and if needed configure webhook (URL, method, headers) or alert templates.  
   - Save the service.

4. **Viewing status and history**  
   - In the external services table, the **Status** column shows current status (healthy / unhealthy / unknown).  
   - Open the service and on the Monitoring tab at the bottom you will see:  
     - **Health Status** — last check, response time, last error.  
     - **Check History** — table of recent checks (time, status, response time, error).

5. **Monitoring-only configuration**  
   On the external services list page, each row has a monitor icon button (**Configure monitoring**) — it opens the same edit dialog with focus on the Monitoring tab.

### 3.2. Via API

Base path: `GET/POST/PUT/DELETE /api/v2/external_services` and by UUID: `/api/v2/external_services/:external_service_uuid`.

**Create or update a service with monitoring (request body fragment):**

```json
{
  "name": "My API",
  "slug": "my-api",
  "url": "https://api.example.com",
  "monitoringEnabled": true,
  "monitoringConfig": {
    "enabled": true,
    "scheduleExpression": "*/5 * * * *",
    "method": "GET",
    "path": "/health",
    "expectedStatusCode": 200,
    "timeout": "10s",
    "notificationType": "none",
    "messages": {
      "unhealthyMessage": "Service {{.service.name}} is unhealthy",
      "recoveryMessage": "Service {{.service.name}} recovered"
    },
    "webhookMethod": "POST"
  }
}
```

**Get current health status:**

```http
GET /api/v2/external_services/:external_service_uuid/health
```

Response includes `status`, `lastCheckAt`, `lastSuccessAt`, `lastError`, `responseTimeMs`, etc.

**Get check history:**

```http
GET /api/v2/external_services/:external_service_uuid/health/history?limit=10&offset=0
```

Response is an array of entries with fields `checkedAt`, `status`, `responseTimeMs`, `error`.

---

## 4. Cron expressions (schedule)

Format: **minute hour day_of_month month day_of_week** (standard cron with step and range support).

Examples:

| Expression | Meaning |
|------------|---------|
| `*/5 * * * *` | Every 5 minutes |
| `*/1 * * * *` | Every minute |
| `0 * * * *` | Every hour (at minute 0) |
| `0 */2 * * *` | Every 2 hours |
| `0 0 * * *` | Once a day at midnight |
| `0 9 * * 1-5` | At 9:00 on weekdays |

An invalid expression will cause a validation error when saving the service (both on the backend and in the UI when checks are enabled).

---

## 5. Limitations and notes

- **Timeout** is the maximum time to wait for a single health-check HTTP request; **schedule (cron)** is how often that check runs. They are independent.
- One check per service runs at most once per cron tick; concurrent checks for the same service are blocked.
- History: **no more than 10 entries per service**; the limit is the same for all services (10 per service), configurable only in code.
- Notifications are sent **only on status change** (e.g. healthy → unhealthy or unhealthy → healthy), not on every check.
- For webhook you can set URL, method, and headers; the body uses the same templates as the messages.

---

## 6. Where things are configured

| What | Where |
|------|-------|
| Enable monitoring, schedule, method, path, status code, timeout, body, headers | UI: external service form → Monitoring tab / API: `monitoringConfig` in create/update body |
| Notification type and webhook/alert parameters | Same Notifications block in the form / same fields in `monitoringConfig` |
| View current status | UI: Monitoring tab on service details; API: `GET .../health` |
| View check history | UI: Check History table on Monitoring tab; API: `GET .../health/history` |
| History limit (10 per service) | Backend code only (constant or config at startup) |

---

## 7. Quick checklist

1. Create or open an external service (URL, system account if needed).  
2. Enable **Enable Monitoring** and **Enable Health Check**.  
3. Set **Schedule (cron)** (e.g. `*/5 * * * *`).  
4. Optionally change Path, Method, Expected Status Code, Timeout.  
5. Optionally enable notifications (Webhook or System Alert) and configure them.  
6. Save.  
7. Monitor status in the table and service details; check history and backend logs if needed.

If status stays **unknown** after saving, wait for the first schedule run (e.g. within 5 minutes for `*/5 * * * *`) or check backend logs for check execution errors.
