---
title: CreateCodeScoringProject
weight: 10
---

CreateCodeScoringProject — creates a new project in CodeScoring.
The action uses the CodeScoring API to register a project with the specified parameters:
- project name,
- repository URL,
- VCS ID,
- an option to automatically run SCA analysis after cloning the repository.

### Request example

```yaml
name: example-project
repository: https://gitlab.example.com/group/project.git
vcs_id: 2
run_sca_after_clone: true
```

### Request specification

| Name                 | Required | Description                                                                 |
| -------------------- | -------- | ---------------------------------------------------------------------------- |
| name                 | Yes      | Project name in CodeScoring                                                  |
| repository           | Yes      | Repository URL (e.g., <https://gitlab.example.com/group/project.git>)        |
| vcs_id               | Yes      | VCS ID in CodeScoring (must be greater than 0)                               |
| run_sca_after_clone  | No       | Automatically run SCA analysis after cloning the repository                  |

### Response

The response contains the created project object with the following information: project identifier (pk), name, project type, description, repository information, project status, access permissions, license, number of dependencies and vulnerabilities, project languages, scan schedule status, and the dates of the first and last SCA scans.
