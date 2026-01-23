---
title: External services monitoring
menuTitle: External services monitoring
d8Edition: ee
moduleStatus: experimental
---

External services monitoring is a mechanism for automatically checking the availability and correct operation of external services connected to the platform. The platform runs periodic health checks and displays the current status of each service in the UI.

## How it works

The platform automatically performs health checks for external services on a schedule. For each service, you can configure the check parameters: request method, path, expected HTTP status, timeout, headers, and (if needed) the request body.

The result of each check is stored, and the service status is updated based on the most recent successful or failed response.

Possible statuses:

- `healthy`: the service is reachable and the response matches the expected parameters.
- `unhealthy`: the service is unreachable, does not respond within the timeout, or returns an unexpected status/error.
- `unknown`: the status is not yet determined (for example, the check has not run yet or there is no data about the last check).

## Configuring monitoring

Monitoring is configured in **Administration** → **External services** when creating or editing an external service.

### Monitoring parameters

| Parameter | Description | Example |
|----------|-------------|---------|
| **Enable monitoring** | Enables or disables automatic health checks for the service | Enabled |
| **Check interval** | How often to run the service health check | `5m`, `1h`, `30s` |
| **Request method** | HTTP method used for the health check | `GET`, `POST`, `PUT` |
| **Check path** | Server path used for the health check | `/health`, `/api/status` |
| **Expected status code** | HTTP status code considered successful | `200`, `201` |
| **Request timeout** | Maximum time to wait for a response | `10s`, `30s` |
| **Headers** | Additional HTTP headers to include in the request | `Content-Type: application/json` |
| **Request body** | Request body (for POST/PUT methods) | JSON or YAML |

### Configuration examples

Health check using a GET request:

- Method: `GET`
- Path: `/health`
- Expected status code: `200`
- Timeout: `10s`

Health check using a POST request:

- Method: `POST`
- Path: `/api/health/check`
- Expected status code: `200`
- Request body: `{"check": true}`
- Headers: `Content-Type: application/json`

### Using templates

You can use templates in request headers and the request body to substitute values:

- `{{.service.name}}`: service name
- `{{.service.url}}`: service URL
- `{{.status}}`: current status
- `{{.error}}`: error message
- `{{.responseTime}}`: response time

## Viewing health status

The health status of an external service is shown in several places:

- in the external services list
- on the service details page
- in the check history
- in the dashboard widget (if enabled)

**In the external services list**: In **Administration** → **External services**, the table includes a **Status** column with color indication:

- Green - the service is operating normally (`healthy`).
- Red - the service is unavailable (`unhealthy`).
- Gray - the status is not determined (`unknown`).

**On the service details page**: When you open an external service, you can see detailed health information:

- current status
- time of the last check
- time of the last successful check
- response time
- last error message (if any)

**In the check history**: The service details page shows the recent health check history, including:

- status of each check
- check time
- response time
- error message (if the check failed)

By default, the platform stores the last 10 history records. You can change this limit in the platform configuration.

**In the dashboard widget**: The **External services health** widget lets you track the state of all external services that have monitoring enabled.

The widget shows:

- Summary statistics: total number of services and how many are in `healthy`, `unhealthy`, and `unknown` states
- Services table: a list of services with the following details:
  - Service name
  - Health status
  - Response time
  - Time of the last check
  - Error description (if any)

The widget supports pagination for easier viewing when there are many services. You can choose how many entries to display per page: 5, 10, 20, 50, or 100.

## Notifications

When an external service health status changes, the platform can send notifications. Notification settings are configured per service in that service’s monitoring configuration.

Notification types:

- **No notifications**: notifications are not sent.
- **Webhook**: sends a notification to the specified webhook URL.
- **System notification**: creates a system notification in the platform (available to administrators only).

Notifications are triggered on the following events:

- `healthy` → `unhealthy`: the service becomes unavailable or starts returning errors.
- `unhealthy` → `healthy`: the service recovers and passes health checks again.

You can use template variables in notification messages:

- `{{service.name}}`: service name
- `{{service.url}}`: service URL
- `{{status}}`: current status
- `{{error}}`: error message
- `{{responseTime}}`: response time

Example message:

```text
Service {{.service.name}} ({{.service.url}}) became unavailable.
Error: {{.error}}
Response time: {{.responseTime}}ms
```

## Availability check before use

The platform can check an external service’s availability before running operations that depend on it:

- **Data sources**: before data synchronization.
- **Actions**: before executing an action that uses the external service.
- **Widgets**: before loading widget data.
- **Widget actions**: before running an action from the widget UI.

If the service is unavailable (`unhealthy`), the operation is blocked and the user sees a clear error message. This reduces the risk of silent failures and prevents starting operations that are unlikely to complete successfully.

> **Warning:** if monitoring is disabled for the service, availability checks before use are not performed, and operations may run regardless of the service’s actual state.

## Recommendations

### Check interval

Choose the interval based on the service’s criticality and the acceptable incident response time:

- **Critical services**: every 1 to 5 minutes.
- **Important services**: every 15 to 30 minutes.
- **Regular services**: once an hour or less frequently.

### Timeout

Set the timeout to account for normal response time and network latency:

- for fast services: 5 to 10 seconds
- for slow services: 30 to 60 seconds

### Use dedicated health-check endpoints

Use dedicated health endpoints (for example, `/health`, `/api/status`). These endpoints should:

- Respond quickly.
- Ideally require no authentication, or use minimal permissions.
- Avoid creating noticeable load on the service.

### Monitoring critical services

For services that key processes depend on, it is recommended to:

- Enable notifications on status changes.
- Configure a short check interval.
- Use system notifications for administrators (or a webhook integration with your on-call channel).
