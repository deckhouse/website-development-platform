---
title: Audit logs
description: Audit log contents, storage, filtering, retention, and CSV export in DDP.
weight: 40
---

Audit logs record all operations that users perform through the Deckhouse Development Platform (DDP) API. The logs are stored in a PostgreSQL database and support auditing of user activity.

## Components

The following components create and store audit logs:

- DDP Backend creates audit log entries.
- PostgreSQL stores the data.

## Audit log contents

Each HTTP request except `GET` creates an audit log entry with the following fields:

- "Email" — email address of the user who made the request.
- "IP" — client IP address.
- "Request body" — HTTP request body.
- "Path" — requested API path.
- "Method" — request HTTP method (`POST`, `PUT`, `DELETE`, or `PATCH`).
- "Status" — response HTTP status code.
- "Date" — request date.

## How it works

DDP Backend automatically creates audit logs through request-processing middleware for all HTTP requests except `GET`.

To reduce database load, entries are buffered with the following settings:

- Batch size — 40 entries.
- Write interval — 20 seconds.
- In-memory buffer size — 4 entries.

## Storage

Audit logs are stored in the PostgreSQL `audit_logs` table. The table is partitioned by day based on the `timestamp` field.

The table structure is maintained automatically. Every day at `00:00` server time, the system prepares space for logs for the next 7 days.

Logs are retained indefinitely and old entries are not deleted automatically. To control retention, configure external tools, such as cron jobs or database cleanup scripts, to delete old log partitions automatically.

## Viewing logs

Audit logs are available in the web interface under "Administration" → "Audit". You can filter logs by the following fields:

- Period, including start and end date and time.
- User email.
- IP address.
- Path.
- Request method.
- Response status.

## Exporting to CSV

The audit log page provides a "Download .csv" button. Export is available only with permission to read audit logs.

The file includes all entries that match the current filters and sorting. Export does not run if more than 100,000 events match.

## Configuration

Audit logging is enabled by default and requires no additional configuration. To control log retention, configure automatic deletion of old database partitions.
