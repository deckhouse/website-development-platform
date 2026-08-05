---
title: CreateNexusRepository
weight: 10
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used as Basic Auth for requests to Nexus.
{{< /alert >}}

`CreateNexusRepository` — creates a new repository of any supported type (maven, docker, npm, etc.) in Nexus Repository Manager 3 using the REST API.  
Format, type, and other key settings are fully configurable and match the Nexus API.

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
| description   | No           | Documentation on the purpose of this action/repository. Not used by Nexus itself, only for the UI          | -                                          |
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

- Only use the blocks (`maven`, `group`, `proxy`, `docker`, etc.) that are supported for your type/format.
- For maven hosted, `maven: {versionPolicy, layoutPolicy}` is required.
- For group, `group.memberNames` is required.
- For proxy, `proxy.remoteUrl` is required.
- For docker, specify the docker-specific fields in `docker`.
