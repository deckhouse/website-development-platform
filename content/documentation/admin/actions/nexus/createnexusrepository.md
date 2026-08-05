---
title: CreateNexusRepository
weight: 10
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

`CreateNexusRepository` — creates a new repository of any supported type (maven, docker, npm, etc.) in Nexus Repository Manager 3 using the REST API.  
The format, type, and other key settings are fully configurable and correspond to the Nexus API.

### Request example (Maven hosted)

```yaml
description: |
  Maven hosted repo for internal Java build artifacts.
name: my-maven-repo
format: maven
type: hosted
online: true
storage:
  blobStoreName: default
  strictContentTypeValidation: true
  writePolicy: ALLOW
cleanup:
  policyNames:
    - maven-cleanup
maven:
  versionPolicy: RELEASE
  layoutPolicy: PERMISSIVE
```

### Request example (Docker group)

```yaml
description: |
  Docker group repo aggregating hosted+proxy.
name: my-docker-group
format: docker
type: group
online: true
storage:
  blobStoreName: default
  strictContentTypeValidation: true
group:
  memberNames:
    - docker-hosted
    - docker-proxy
docker:
  v1Enabled: false
  forceBasicAuth: true
  httpPort: 5001
```

### Request specification

| Field         | Required     | Description                                                                                            | Example                                    |
| ------------- | ------------ | -------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| description   | No           | Description of the action or repository's purpose. Used only in the UI, not by Nexus itself                  | -                                          |
| name          | Yes          | Name of the repository to create. Must be unique within Nexus                                               | my-maven-repo                              |
| format        | Yes          | Format (`maven`, `docker`, `npm`, `raw`, etc.)                                                              | maven                                      |
| type          | Yes          | Type: `hosted`, `proxy`, or `group`                                                                        | hosted                                     |
| online        | Yes          | Whether the repository is available (`true`/`false`)                                                       | true                                       |
| storage       | Yes          | Storage object: `blobStoreName`, `strictContentTypeValidation`, `writePolicy`                               | [Example](#request-example-maven-hosted)   |
| cleanup       | No           | Linked cleanup policies (`policyNames`)                                                                     | policyNames: [maven-cleanup]               |
| maven         | For maven    | Maven-only: `versionPolicy`, `layoutPolicy`                                                                 | [Example](#request-example-maven-hosted)   |
| proxy         | For proxy    | Proxy repository: `remoteUrl`, `contentMaxAge`, `metadataMaxAge`                                            | -                                          |
| group         | For group    | List of `memberNames` values                                                                                | [Example](#request-example-docker-group)   |
| docker        | For docker   | Docker-specific parameters: `httpPort`, `v1Enabled`, `forceBasicAuth`                                       | [Example](#request-example-docker-group)   |
| component     | Very rare    | Only for certain non-standard scenarios                                                                     | -                                          |
| attributes    | No           | Any custom fields                                                                                            | -                                          |

### Requirements

- Use only the blocks (`maven`, `group`, `proxy`, `docker`, etc.) supported by the selected type and format.
- For Maven hosted repositories, `maven: {versionPolicy, layoutPolicy}` is required.
- For group repositories, `group.memberNames` is required.
- For proxy repositories, `proxy.remoteUrl` is required.
- For Docker repositories, specify the Docker-specific fields in `docker`.
