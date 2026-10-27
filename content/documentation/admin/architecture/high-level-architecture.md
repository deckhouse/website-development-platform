---
title: High-level architecture
description: Purpose of Deckhouse Development Portal, user groups, and ways to access the portal.
weight: 5
---

## System goal and purpose

Deckhouse Development Portal (DDP, portal) is an IDP (Internal Developer Platform), an internal developer portal that brings together tools, infrastructure services, and development processes.

The portal provides a single layer for interacting with infrastructure and engineering services and is used to automate operations related to the product lifecycle.
DDP provides unified access to tools and resources, formalizes and automates development processes, and simplifies onboarding of new users and teams.

The main tasks of the portal are as follows:

- Enforcing common development standards across teams through templates and standard configurations
- Managing dynamic environments with automatic creation, update, and deletion of environments
- Integrating with external infrastructure services through APIs
- Simplifying developer and team onboarding with templates and documentation
- Digitizing development processes and workflows through a BPMN-like process description mechanism
- Providing a built-in CMDB that automatically collects data from infrastructure services and visualizes dependencies
- Controlling entity properties through status checks

## System users

Several groups of users work with the portal.

### Development teams

Development teams use DDP to work with portal entities and resources.
These teams create and manage entities, run actions, view data from datasources, use widgets, and work with processes.

### Management

Management receives statistics and data for decision-making.
Management-level users can view dashboards with metrics, analyze development efficiency, track key indicators, and get reports on team performance.

### Infrastructure teams

These teams provide services and resources through DDP.
They configure integrations with external services, set up resources, create templates and automations, manage environments, and keep the portal running.

### Security teams

Security teams use DDP to provide and manage security tools, perform audits, and monitor compliance.
In the portal, they configure security policies and access permissions, view audit logs, and manage protection mechanisms.

### Administrators

Administrators are users with full permissions to configure and maintain the portal.
They manage user accounts, configure roles and access permissions through RBAC, configure integrations with external services, manage credentials, resources, automations, and portal settings, and view audit logs.

Access permissions for all user groups are configured through role-based access control (RBAC), which provides flexible permission management at the level of global roles, resource roles, and entity roles.

## System access

Users and external systems access the portal through the web interface and the REST API.

### Web interface

All user groups access the portal through the DDP Frontend web interface, which is designed for working with the portal in a web browser.

Access protocol: HTTPS (TCP/443)

Authentication: Users authenticate through the Dex component, which integrates with external authentication providers (Keycloak, LDAP, and others) through standard protocols (OIDC, LDAP).
After successful authentication, the user receives a JWT access token, which is used to authorize requests to the portal.

### REST API

Administrators and developers can access DDP Backend directly through the REST API to interact with the portal programmatically.

Access protocol: HTTPS (TCP/443)

The browser authenticates with a session stored in a cookie file with the `HttpOnly` flag (JWT from Dex).
For programmatic access, use an API token from the "Profile" section, passed in the `Authorization: Bearer <API_TOKEN>` header.

The REST API provides a full set of functions for managing all portal objects: entities, resources, actions, datasources, widgets, processes, and other components.

## Access from outside and inside the company perimeter

DDP is deployed in a Kubernetes cluster and can be accessible both inside the corporate perimeter and from outside, depending on the network infrastructure configuration.

### Access inside the perimeter

In a typical scenario, users access the portal through the company's internal network. This type of access has the following characteristics:

- The web interface and REST API are available through an internal Ingress controller.
- Access is provided through internal domain names (for example, `ddp.internal.company.com`).
- Authentication can be integrated with corporate LDAP or Active Directory through Dex.
- Additional protection can be provided by Kubernetes network policies.

### Access outside the perimeter

In a typical scenario, users access the portal from outside the corporate network. This type of access has the following characteristics:

- The web interface and REST API are available through a public Ingress controller.
- Access is provided through public domain names (for example, `ddp.company.com`).
- Authentication can be integrated with external authentication providers through Dex.

### Hybrid access

The portal can be configured to support both scenarios at the same time:

- Internal users access the portal through an internal Ingress.
- External users (for example, remote employees) access the portal through a public Ingress or VPN.
- Different user groups can use different authentication providers through Dex.

The administrator configures access when deploying the portal, using Ingress controller parameters and Kubernetes network policies.
