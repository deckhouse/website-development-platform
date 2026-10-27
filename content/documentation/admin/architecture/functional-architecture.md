---
title: Functional architecture
description: Deckhouse Development Portal components, their functions, and network interactions.
weight: 10
---

{{< alert level="info" >}}
The diagram uses the following simplifications.
It shows containers of different pods interacting with each other directly.
In fact, they interact through the corresponding Kubernetes Services (internal load balancers).
Service names are omitted when they are obvious from the context.
In other cases, the Service name is shown above the arrow.
Pods can run in multiple replicas, but the diagram shows each pod as a single replica.
{{< /alert >}}

The following diagram shows the Deckhouse Development Portal (DDP, portal) architecture at level 2 of the C4 model and its interaction with other components:

![DDP architecture](../images/functional-architecture/c4-l2-ddp.svg)

## Components

DDP consists of the following components:

1. **Frontend** is the portal web interface built with Vue.js 3 and TypeScript.
   The component consists of a single **frontend** container and provides users with a graphical interface for working with the portal in a web browser.

    Main functions:

    * Rendering the user interface for managing services, environments, and resources
    * Handling user actions in the browser and sending requests to the backend through the REST API

    Technical details:

    * It serves the web interface files to the browser through the Ingress controller or Gateway
    * It does not access the backend itself: the user's browser sends REST API requests through the same Ingress controller or Gateway

1. **Backend** is the server side of the portal, written in Go.
   The component provides a REST API for managing all portal mechanisms.

    Main functions:

    * Providing the REST API for programmatic interaction with the portal
    * Processing the portal business logic
    * Implementing access control (RBAC) and user permission management
    * Integrating with external infrastructure services to perform various operations
    * Managing data through PostgreSQL and Redis

    Technical details:

    * Accepts API requests from the web interface in the user's browser and from external REST API clients
    * Signs users in through Dex and validates access tokens locally using cached signing keys
    * Assigns tasks to the agent and forwards its requests to external language models
    * Connects to Redis to coordinate the task queue
    * Connects to PostgreSQL to store and manage data
    * Connects to ClickHouse, if configured, to read the data stored there

    It consists of the following containers:

    * **wait-for-postgres**: Init container that waits until PostgreSQL starts accepting connections. For managed PostgreSQL instance, it also runs initial SQL scripts, for example, to create the required PostgreSQL extensions.
    * **wait-for-redis**: Init container that waits until Redis starts accepting connections.
    * **wait-for-clickhouse**: Init container that waits until ClickHouse (if configured) starts accepting connections. For managed ClickHouse instance, it also runs initial SQL scripts.
    * **migrations**: Init container that updates the database schema when the component starts.
    * **backend**: Main container.

1. **Worker** is a component written in Go for running background tasks and asynchronous operations.
   Worker processes tasks from Redis Streams queues: it runs actions and processes and synchronizes datasources.
   For details, see ["Workers"](workers/) section.

    Main functions:

    * Synchronizing data from external sources
    * Running actions and processes asynchronously
    * Running entity status checks
    * Running AI agents from "AI agent" process elements

    Technical details:

    * Can be deployed in multiple instances for scalability
    * Uses Redis Streams to receive tasks from queues
    * Connects to PostgreSQL to read and update data
    * Integrates with external infrastructure services to perform operations
    * Assigns tasks to the agent when running "AI agent" process elements
    * Moves data from PostgreSQL to ClickHouse, if configured

    It consists of the following containers:

    * **wait-for-postgres**: Init container that waits until PostgreSQL starts accepting connections. For managed PostgreSQL instance, it also runs initial SQL scripts, for example, to create the required PostgreSQL extensions.
    * **wait-for-redis**: Init container that waits until Redis starts accepting connections.
    * **wait-for-clickhouse**: Init container that waits until ClickHouse (if configured) starts accepting connections. For managed ClickHouse instance, it also runs initial SQL scripts.
    * **worker**: Main container.

1. **Agent** is the component that implements the [platform agent](../ai-agents/platform-agent/).
   The component consists of a single **agent** container.
   By default, the agent handles requests in the [AI assistant](../../user/ai-assistant/) and in the ["AI chat"](../widgets/ai/chat/) widget; you can also select it in the ["AI agent"](../processes/overview/#ai-agent) process element.

    Technical details:

    * Accepts tasks from the backend and worker over the A2A protocol
    * Accesses external language models and portal tools through the backend
    * Available only inside the cluster, without an Ingress controller or Gateway

    DDP also supports connecting an [external AI agent](../ai-agents/external-agents/).

Short-term and long-term data storage is provided by Redis, PostgreSQL, and ClickHouse.

{{< alert level="info" >}}
For each of these components, you select a mode during installation: `internal` (an instance that is part of DDP), `managed` (an instance created by the corresponding Deckhouse managed module), or `external` (an existing external instance).
For production, use external database instances.
For details, see [Installation](../install/).
{{< /alert >}}

The data storage components perform the following functions:

1. **Redis** is an in-memory data store that the portal uses for task queues.
   In `managed` mode, the Redis-compatible Valkey is used instead of Redis.

    Main functions:

    * Implementing task queues through Redis Streams for asynchronous processing
    * Supporting distributed locks to coordinate multiple worker instances
    * Delivering events generated by entity changes to automation handlers.
      The events themselves are stored in PostgreSQL.

    Technical details:

    * Used to store task queues that are processed by the worker

1. **PostgreSQL** is the main relational database of the portal, used to store all persistent information.

1. **ClickHouse** is a store for large volumes of data, for example, the history of entity property changes.

    {{< alert level="info" >}}
    You can install ClickHouse as part of DDP, create it using the `managed-clickhouse` module, or connect an external instance.
    The portal works without ClickHouse, but features that depend on it are unavailable.
    For example, the history of entity property changes is not saved.
    For details, see [ClickHouse](../install/#clickhouse).
    {{< /alert >}}

    Main functions:

    * Storing large volumes of records
    * Serving queries that select from this data

    Technical details:

    * Populated by the worker, which moves data to it from PostgreSQL
    * Accepts backend and worker connections over the native ClickHouse protocol

The following component is not part of DDP:

1. **Dex**: An authentication and authorization component that acts as an identity provider.
   Dex provides a single sign-in point for users and integrates with various external authentication providers.

    {{< alert level="info" >}}
    The Dex component is part of the [`user-authn`](/modules/user-authn/) module in Deckhouse Platform (DP).
    {{< /alert >}}

    Main functions:

    * Managing the user authentication process
    * Issuing access tokens
    * Integrating with external authentication providers (Keycloak, LDAP, and others)
    * Providing signing keys for token verification

    Technical details:

    * Shows the sign-in page to the user
    * Issues and refreshes tokens on backend requests
    * Periodically provides token signing keys for verification
    * Integrates with external authentication providers through standard protocols (OIDC, LDAP, and others)

## Workspaces

[Workspaces](../workspaces/overview/) are created in the cluster and namespace of the template.
When installed by the module, OpenTofu operation Jobs run in the module namespace.
Persistent workspace data is stored on separate volumes; the OpenTofu state is passed to the portal HTTP backend.

The starter template creates a pod with the following containers:

| Container | Purpose |
| --- | --- |
| `bootstrap` | Init container: It clones the repository, delivers files and skills, and runs scripts |
| `runtime` | OpenCode agent that runs tasks from the chat and processes |
| `ide` | Code-server IDE with files from the same persistent volume |
| `gate` | Checks IDE access through the portal |

The browser opens the IDE on a separate HTTPS domain through the Ingress of the target cluster.
The `gate` container checks access in DDP.
The agent accesses the model and MCP through the portal, so the pods require access to the DDP HTTPS address.
The setup is described in the [network and storage examples](../workspaces/network-storage/).

## Network interactions

![DDP components network interactions](../images/functional-architecture/ddp-network-flows.ru.svg)

### Incoming connections

Incoming connections are accepted by the cluster Ingress controller or Gateway over HTTPS (TCP/443) and forwarded to the portal components on port TCP/8080:

* Users → DDP Frontend: The browser loads the web interface files.
* Users → DDP Backend: The browser sends REST API requests from the web interface; external clients access the REST API directly.
  Requests to portal webhooks also go to the backend.
* Users → Dex (TCP/443): The browser opens the Dex sign-in page.

### Interaction between portal components

The portal components interact with each other as follows:

* DDP Backend → Dex (TCP/443): Backend obtains user tokens at sign-in and on refresh, as well as signing keys for token verification.
* DDP Backend ↔ Redis (TCP/6379): Backend uses Redis to coordinate task queues and store events.
* DDP Backend ↔ PostgreSQL (TCP/5432): Backend reads and writes persistent data.
* DDP Worker ↔ Redis (TCP/6379): Worker receives tasks from Redis Streams queues and uses distributed locks.
* DDP Worker ↔ PostgreSQL (TCP/5432): Worker updates data in the database after running tasks.
* DDP Backend ↔ ClickHouse (TCP/9000, optional): Backend reads the data stored in ClickHouse.
* DDP Worker ↔ ClickHouse (TCP/9000, optional): Worker moves data from PostgreSQL to ClickHouse and deletes from it the data of objects that have already been deleted in the portal.
* DDP Backend → DDP Agent (TCP/8080): Backend assigns tasks from the AI assistant and the "AI chat" widget to the agent.
* DDP Worker → DDP Agent (TCP/8080): Worker assigns tasks from "AI agent" process elements to the agent.
* DDP Agent → DDP Backend (TCP/8080): Agent sends requests to the language model and calls portal tools.

### Integration with external services

The portal components interact with external services as follows:

* Dex ↔ Authentication providers (Keycloak, LDAP, and others): Dex integrates with external user management systems.
* DDP Backend ↔ Infrastructure services (TCP/443): Backend interacts with external services to get information.
* DDP Worker ↔ Infrastructure services (TCP/443): Worker performs operations with external services when synchronizing data and running actions.
* DDP Backend → External language models (TCP/443): Backend forwards agent requests to the language model provider.
* DDP Backend and DDP Worker → External AI agents (TCP/443): The portal assigns tasks to connected external agents over the A2A protocol.
