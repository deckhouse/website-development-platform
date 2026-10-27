---
title: Automatic service upgrades
weight: 50
description: Configuring DDP resources, a relation, a datasource, actions, a process, and an automation that create merge requests in services when a new template version is released.
---

This guide describes how to configure an automatic mechanism that upgrades all services created from a template when a new version of that template is released.
A new template version is detected by a new tag added to the template's Git repository.

{{< alert level="info" >}}
The names of resources, properties, actions, and processes in this guide are examples and may differ depending on your settings.
{{< /alert >}}

## System components and their purpose

Automatic service upgrades use the following components of Deckhouse Development Portal (DDP):

- Resources: Elements of the DDP catalog data model that define how information is stored.
  Each resource can contain properties for storing entity metadata, as well as relations to other resources for displaying dependencies.

  This guide uses two resources:

  - "Templates": Stores information about template repositories (repository URL and ID, latest version tag).
  - "Services": Stores information about created services (repository URL and ID, templating variables, default branch).

- Relation: Defines a dependency between resources and makes it possible to automatically find all services related to a specific template.

- [Datasource](../datasources/overview/): A DDP mechanism that synchronizes information from external infrastructure systems (for example, GitLab) to the portal catalog.
  Datasources periodically poll external systems, retrieve up-to-date information, and update the corresponding entities in the catalog.
  In this guide, the datasource:

  - Synchronizes information about template repositories from GitLab.
  - Updates the list of tags for each template.
  - Automatically updates the "Latest tag" property in entities of the "Templates" resource.

- Action: A DDP mechanism that lets portal users interact with external infrastructure services via an API.
  Actions can create projects in GitLab, create secrets in secret management systems, deploy applications, and perform other operations.

  This guide uses three actions:

  - Creating a new project in GitLab to host the service.
  - Cloning code from the template repository to the new service repository, applying templating variables.
  - Creating a merge request in the service repository with changes from the new template version.

- Process: An automation mechanism for complex business processes that lets you build visual execution diagrams for actions, with support for sequential or parallel execution.
  Processes combine multiple actions into a single execution chain.
  In this guide, the process:

  - Runs the creation of a project in GitLab and then the creation of a repository from the template, in sequence.
  - Provides the user interface for creating a service from a template.

- Automation: A mechanism by which the portal responds to changes in entity specifications.
  Automations subscribe to update events of resource entities based on specific rules (triggers) and run actions, processes, or datasource synchronization.
  In this guide, the automation:

  - Tracks changes to the "Latest tag" property in entities of the "Templates" resource.
  - When a new tag is detected, runs the "Upgrade service from template" action for all related services.
  - Creates a merge request in each service repository with changes from the new template version.

Automatic upgrade diagram:

![Automatic update diagram](../images/service-autoupdates/workflow.png)

How it works:

1. The "Datasource" periodically synchronizes information about templates from GitLab and updates the property with the `repository_last_tag` identifier in entities of the "Templates" resource.
1. The "Automation" tracks changes to the property with the `repository_last_tag` identifier using triggers.
1. When a new tag is detected, the automation runs the "Upgrade service from template" action for all services related to the updated template.
1. The "Action" creates a merge request in each service repository that includes changes from the new template version.

As a result, when a new tag is added to a template repository, all related services automatically receive an upgrade proposal as a merge request.

## Configuring components

The following sections describe the configuration of each system component.
If you use different names and property identifiers in your resources, use them in all actions, automations, and datasources that reference these properties.

This guide covers:

1. Required resources in DDP and their properties.
1. Required relations between resources.
1. Required [datasources](../datasources/overview/) for retrieving information about new template versions.
1. Required [actions](../actions/overview/) for creating a service from a template.
1. A required process for running a chain of [actions](../actions/overview/) in sequence.
1. A required [automation](../automations) that implements the following functionality:
    - DDP automatically checks for a new template version.
    - DDP automatically creates a merge request with changes from the new version in services created from the template.

## Creating resources and a relation

"Resources" are elements of the DDP catalog data model that define how information is stored.
Each resource can contain properties for storing entity metadata, as well as relations to other resources for displaying dependencies.

{{< alert level="warning" >}}
Make sure the property identifiers are specified correctly in all actions, automations, and datasources that reference these properties.
{{< /alert >}}

### "Templates" resource

You must have a resource for storing information about template repositories.
In this guide, it is named "Templates", but you can use any name you prefer.
If the resource does not exist yet, [create it](#creating-a-resource).

The resource must contain the following properties for storing data about template repositories:

- The "URL" property of the URL type: Stores the URL of the template repository in GitLab.
  This parameter is required for actions that work with the template repository.

- The "Repository ID" property of the Number type: Stores the internal repository ID in GitLab.
  It is used by actions that require a GitLab project ID.

- The "Latest tag" property of the String type: Contains the name of the latest tag (version) of the template.
  The automation uses this parameter to track new template versions.
  Each time the datasource updates this property, the automation checks for a change and starts upgrading the related services.

If the properties are missing, open the resource for editing and add them on the resource's "Properties" tab by clicking "Add property".

### "Services" resource

You must have a second resource for storing information about created services.
In this guide, it is named "Services", but you can choose any name you prefer.
If the resource does not exist yet, [create it](#creating-a-resource).

The resource must contain the following properties for storing metadata about created services:

- The "URL" property of the URL type: A link to the service repository in GitLab.
  Actions use it to work with the service repository.

- The "Repository ID" property of the Number type: The internal repository ID in GitLab.
  It is required for actions that require a GitLab project ID.

- The "Variables used for templating" property of the YAML type: Contains the variables that were applied when cloning code from the template to the service.
  These variables are required when upgrading the service from the template because they must be applied again to correctly render new changes from the template.

- The "Default branch" property of the String type: The name of the repository's main branch (for example, `main` or `master`).
  The service upgrade action uses it to determine the target branch of the merge request.

If the properties are missing, open the resource for editing and add them on the resource's "Properties" tab by clicking "Add property".

### Creating a resource

To create resources, go to the "Catalog" section.
Create a new resource.
To do this, click "Create resource" at the bottom of the catalog sidebar.

A window for creating a new resource opens, with several tabs.

#### "Main information" tab

Fill in the following fields on the "Main information" tab:

- "Name": [Specify the resource name].

- "Owner": [Specify the owner].

- "Owning team": [Specify the owner team].

#### "Properties" tab

Go to the "Properties" tab and add the required resource properties.
To add a new parameter, click "Add property".

Example of the "Templates" resource properties:

| Name          | Identifier          | Description                               | Type   |
|---------------|---------------------|-------------------------------------------|--------|
| URL           | `web_url`           | Link to the service repository            | URL    |
| Repository ID | `project_id`        | Internal repository ID in GitLab          | Number |
| Latest tag    | `repository_last_tag` | Latest tag created in the Git repository | String |

Example of the "Services" resource properties:

| Name                          | Identifier          | Description                                                  | Type   |
|-------------------------------|---------------------|--------------------------------------------------------------|--------|
| URL                           | `web_url`           | Link to the service repository                               | URL    |
| Repository ID                 | `project_id`        | Internal repository ID in GitLab                             | Number |
| Variables used for templating | `template_values`   | Variables that were applied when cloning from the template   | YAML   |
| Default branch                | `default_branch`    | Name of the repository's main branch                         | String |

### Relation

You must have a relation that reflects the dependency between templates and the services created from them.
If the relation does not exist yet, [create it](#creating-a-relation).

In the relation, the resource that stores information about templates must be selected as the parent resource ("Parent resource"),
and the resource that stores information about services must be selected as the child resource ("Child resource").

### Creating a relation

To create a relation, click "Create relation" at the bottom of the "Catalog" section sidebar.

A window for creating a new relation opens.
Fill in the data according to the table below:

| Parameter       | Value                                                  |
|-----------------|--------------------------------------------------------|
| Name            | [Enter the relation name, for example, `Templates-Services`] |
| Parent resource | [Select the name of the resource with templates]       |
| Child resource  | [Select the name of the resource with services]        |

## Datasource

You must have a datasource that automatically retrieves information about the current state of templates from GitLab.
If the datasource does not exist yet, [create it](#creating-a-datasource).

### Checking the datasource

On the "Main information" tab, the "Templates" resource must be selected in the "Resource" field.

On the "Backend" tab, check that automatic synchronization is enabled.
The "Periodic synchronization" parameter must be enabled so that information about templates is updated automatically when templates change in GitLab.
The "Launch frequency" parameter must contain a schedule in CRON format, for example, `1 * * * *` to run every hour.
The schedule determines how often the datasource checks template repositories for new tags.

The following keys must be set in the backend parameters:

| Key         | Value                     | Description                                                     |
|-------------|---------------------------|-----------------------------------------------------------------|
| `group_ids` | [ID of the template group] | ID of the GitLab group that contains the template repositories |
| `tags`      | `true`                    | Retrieval of tag information for each repository is enabled     |

For details on the available parameters, see [GitlabProjects](../datasources/types/#gitlabprojects).

On the "Rules" tab, rules must be configured to update the "Templates" resource properties with data retrieved from GitLab:

- The "URL" property stores the URL of the template repository in GitLab.
  `{{ .web_url }}` must be used as the source.

- The "Repository ID" property stores the internal repository ID in GitLab.
  `{{ .id }}` must be used as the source.

- The "Latest tag" property stores the latest tag of the Git repository (in this guide, it is used as the template version).
  For example, you can use the `{{ ( index ( .ddp_repository_tags ) 0 ).name }}` source: this expression gets the latest tag created in the Git repository by taking the first element of the tag array.
  If needed, you can configure a different way of getting the tag.
  This property is critical for the automation because a change in its value triggers service upgrades.

The source identifiers are taken from the [GitLab Projects API](https://docs.gitlab.com/api/projects/#get-a-single-project) specification.

### Creating a datasource

To create [datasources](../datasources/overview/), go to "Self-service" → "Datasources".
Click "Create" in the upper right corner.

A window for creating a new [datasource](../datasources/overview/) opens, with several tabs.

#### "Main information" tab

Fill in the following fields:

- "Name": Specify the datasource name (for example, `Service templates`).

- "Resource": Select the resource that contains information about templates.

- "Owner": Assign an owner of the datasource.

- "Owning team": Specify the team responsible for the datasource.

#### "Backend" tab

On the "Backend" tab, configure the connection to GitLab:

- "Backend": Select the built-in `GitlabProjects` datasource type for working with GitLab projects.

- "Periodic synchronization": Enable automatic synchronization so that information about templates is updated automatically when templates change in GitLab.

- "Launch frequency": Specify a schedule in CRON format (for example, `1 * * * *` to run every hour).
  The schedule determines how often the datasource checks template repositories for new tags.

- "URL": Specify the URL of your GitLab instance (for example, `https://gitlab.example.com`).

In the backend parameters, configure the GitLab API request parameters:

| Key         | Value                     | Description                                                          |
|-------------|---------------------------|----------------------------------------------------------------------|
| `group_ids` | [ID of the template group] | Specify the ID of the GitLab group that contains the template repositories |
| `tags`      | `true`                    | Enables retrieval of tag information for each repository             |

For details on the available parameters, see [GitlabProjects](../datasources/types/#gitlabprojects).

#### "Rules" tab

On the "Rules" tab, configure the logic for creating and updating entities:

- "Create new entities": Enable the creation of new entities.
  DDP will automatically create records for new templates if a template repository is found in GitLab but does not exist in the catalog yet.

- "Owner of created entities": Assign an owner for new entities.

- "Owning team of created entities": Specify the owner team for new entities.

In the creation rules, configure the identifier and name of the new entity in the portal.
In the "Identifier" field, set the identifier of the entity being created.
For example, the `{{ replaceChar .path_with_namespace "/" "-" }}` expression transforms the path with the namespace, replacing slashes with hyphens to get a unique ID.
In the "Name" field, set the name of the entity being created.
For example, the `{{ .name }}` expression uses the project name from GitLab.

If needed, you can define your own rules based on Go templates.

In the matching rules, configure how existing entities are found during synchronization.
In the "Identifier" and "Name" fields, set the identifier and name of the entity being updated.
For example, you can use the same `{{ replaceChar .path_with_namespace "/" "-" }}` and `{{ .name }}` expressions as in the creation rules.

If needed, you can define your own rules based on Go templates.
If you do not plan to create resource entities manually, use the same rules for matching and creation.

In the update rules, configure updating the "Templates" resource properties with data retrieved from GitLab:

- The "URL" property: The URL of the template repository in GitLab is used as the value.
  Use `{{ .web_url }}` as the source.

- The "Repository ID" property: The internal repository ID in GitLab is used as the value.
  Use `{{ .id }}` as the source.

- The "Latest tag" property: The Git repository tag is used as the value.
  For example, you can use the `{{ ( index ( .ddp_repository_tags ) 0 ).name }}` source: this expression gets the latest tag created in the Git repository by taking the first element of the tag array.
  If needed, you can configure a different way of getting the tag.
  This property is critical for the automation because a change in its value triggers service upgrades.

The source identifiers are taken from the [GitLab Projects API](https://docs.gitlab.com/api/projects/#get-a-single-project) specification.

#### "Authorization" tab

On the "Authorization" tab, configure the credentials and headers required to connect to GitLab.

Configure the credentials.
In the "Identifier" field, specify `token`, and in the "Credentials" field, select the credential that contains a GitLab token with access to the repositories.
The token must have permissions to read projects and tag information in the specified GitLab group.

Configure the header. In the "Key" field, specify `Private-Token`, and in the "Value" field, specify `{{ .credentials.token }}`.
In this case, the value configured for the credentials is used.
If you use an identifier other than `token` for the credentials, replace `token` with your identifier.

For details on the available headers, see [GitlabProjects](../datasources/types/#gitlabprojects) and [System account](../datasources/overview/#system-account).

## Creating actions

Configure three actions that are responsible for creating a service from a template and upgrading services when new template versions are released.

{{< alert level="info" >}}
In the entity update rules, in the `entity parameter` field, specify the identifier of the corresponding property of the resource that stores information about services.
The request bodies use the user form field identifiers given in this guide.
If you set different identifiers, change them in the request body as well.
{{< /alert >}}

### "Create GitLab project" action

You must have an action that initializes a new project in GitLab.
It runs first when a new service is created and creates the GitLab project that will host the service created from the template.
If the action does not exist yet, [create it](#creating-an-action).

Check that on the "Main information" tab, the resource that contains information about services is selected in the "Resource" field.
On the "Update" tab, the "Entity property updates" option must be enabled and update rules must be configured
that write the ID, URL, and default branch of the newly created project to the entity for which the action was run.

Example of entity update rules:

- The "Repository ID" property: The `{{ .response.id }}` source is used. It saves the ID of the created project from the GitLab API response to the entity property.

- The "URL" property: The `{{ .response.web_url }}` source is used. It saves the GitLab URL of the created project to the entity property.

- The "Default branch" property: The `{{ .response.default_branch }}` source is used. It saves the default branch name to the entity property.

The source identifiers (`id`, `web_url`, `default_branch`) are taken from the [GitLab Projects API](https://docs.gitlab.com/api/projects/#get-a-single-project) specification.

### "Create repository from template" action

You must have an action that clones code from the template repository to the new service repository, applying templating variables.
This action runs after the project is created in GitLab and populates it with code from the template.
If the action does not exist yet, [create it](#creating-an-action).

Check that the action has the following settings:

- On the "Main information" tab, the resource that contains information about services is selected in the "Resource" field (the action runs for the entity of the service being upgraded).

- On the "User form" tab, there is a "Template repository ID" parameter with the `template_ddp_slug` identifier (or another one) of the Entities type. It is automatically populated with the DDP identifier of the template entity from which the code will be cloned.
  In the additional settings, specify: Resource: "Templates", Relation: "Templates-Services", Field: "Identifier".
  This lets you select the required template from the related entities.
  You can make this parameter hidden.

- On the "Update" tab, the "Entity property updates" option is enabled and saving the variables used for templating back to the entity for which the action was run is configured.

For example, in the entity update rule for the "Variables used for templating" property, use the `{{ toYAML .response.values.merged }}` source.
This expression saves the variables in YAML format.
These variables are critical for future service upgrades from the template because they must be applied again.
The data is received as a response to a successful action run.

On the "Update" tab, the "Creating relations for the entity" option is also enabled so that the action automatically creates a relation between the created service and its template:

- "Resource": The "Services" resource is selected.

- "Relation": The "Templates-Services" relation is selected.

- "Parent entity slug": Use `{{ .property.template_ddp_slug }}`, the template identifier from the action parameter.

  {{< alert level="warning" >}}
  If the parameter identifier is not `template_ddp_slug`, replace it with the correct one.
  {{< /alert >}}

- "Child entity slug": Leave it empty. The current entity for which the action runs will be used.

### "Upgrade service from template" action

You must have an action that the automation uses to create a new merge request in the service repository.
This merge request will contain changes from the new version (new tag) of the related template.
If the action does not exist yet, [create it](#creating-an-action).

Check that on the "Main information" tab, the resource that contains information about services is selected in the "Resource" field.

Check that the following fields exist on the "User form" tab:

- The "Variables used in templating" parameter with the `template_values` identifier (or another one) of the String type: Stores the variables that were used during the initial templating (they are stored in the entity property).
  Default value: `{{ .entity.properties.template_values }}`. It uses the saved variables from the entity property.
  If the parameter has a different name (for example, `template_vars`), use the corresponding identifier.

- The "Version" parameter with the `version` identifier (or another one) of the Entities type: Gets the value of the new tag (version) from the template entity.
  In the additional settings, specify: Resource: "Templates", Relation: "Templates-Services", Field: "Property", Property: "Latest tag".

- The "Target repository" parameter with the `target_project_id` identifier (or another one) of the String type: The ID of the service repository in GitLab in which the merge request is created.
  Default value: `{{ .entity.properties.project_id }}`. It uses the project ID from the entity property.
  If the property that stores the internal GitLab project ID has a different identifier, use it instead of `project_id`.

- The "Template repository" parameter with the `source_project_id` identifier (or another one) of the Entities type: The ID of the template repository in GitLab.
  In the additional settings, specify: Resource: "Templates", Relation: "Templates-Services", Field: "Property", Property: "Repository ID".

- The "Target repository branch" parameter with the `target_repo_branch` identifier (or another one) of the String type: The main branch of the service repository (for example, `master` or `main`) that the merge request targets.
  If needed, you can specify any other branch.
  Default value: `{{ .entity.properties.default_branch }}`. It uses the default branch from the entity property.
  If the property has a different identifier, use it instead of `default_branch`.

On the "Request configuration" tab, the request body must be configured approximately as follows:

```yaml
merge_request_spec:
  source_branch: update-to-{{ .version }}
  target_branch: {{ .target_repo_branch }}
  title: update-to-{{ .version }}
source_project_tag: {{ .version }}
source_project_id: "{{ .source_project_id }}"
target_project_id: "{{ .target_project_id }}"
values: {{ .template_values | nindent 4 }}
```

In the request body:

- `source_branch`: The name of the branch that contains the changes.
  The example uses a branch whose name includes the tag version.
- `target_branch`: The target branch of the service that the merge request targets.
- `title`: The merge request title, including the version.
- `source_project_tag`: The template version tag from which the changes are taken.
- `source_project_id` and `target_project_id`: The IDs of the template and service repositories.
- `values`: The templating variables, applied again to render the changes correctly.

For details on the built-in backend settings, see [CreateGitlabMergeRequest](../actions/gitlab/creategitlabmergerequest/).

### Creating an action

To create an [action](../actions/overview/), go to "Self-service" → "Actions".
Create a new action.
To do this, click "Create" in the upper right corner.
A window for creating a new [action](../actions/overview/) opens, with several tabs.

#### "Main information" tab

Fill in the following fields on the "Main information" tab:

- "Name": [Specify the action name].

- "Resource": Select the "Services" resource.

- "Owner": [Specify the owner].

- "Owning team": [Specify the owner team].

#### "User form" tab

On the "User form" tab, configure the fields to fill in when running the action.
The set of parameters depends on the action being created.

For the "Create GitLab project" action, add the following parameters:

- The "Name" parameter with the `name` identifier (or another one if needed) of the String type: The name of the new project.
  Default value: `{{ .entity.name }}`. It uses the entity name in DDP.

- The "Path" parameter with the `path` identifier (or another one if needed) of the String type: The URL path to the repository in GitLab.
  Default value: `{{ .entity.slug }}`. It uses the entity identifier in DDP.

- The "Repository group" parameter with the `group` identifier (or another one if needed) of the String type: The ID of the GitLab group in which the service will be created.
  In the "Default value" field, specify the ID of the GitLab group in which the project will be created.

For the "Create repository from template" action, add the following parameters:

- The "Template repository ID" parameter with the `template_ddp_slug` identifier (or another one if needed) of the Entities type: The DDP identifier of the template entity from which the code will be cloned.
  In the additional settings, specify: Resource: "Templates", Relation: "Templates-Services", Field: "Identifier".
  This lets you select the required template from the related entities.

- The "Service repository URL" parameter with the `service_git_url` identifier (or another one if needed) of the URL type: The URL of the repository to which the rendered template will be pushed.
  Default value: `{{ .entity.properties.web_url }}`. It uses the URL of the service entity from the catalog.
  If the URL property in the resource has a different name, use the corresponding identifier (for example, `{{ .entity.properties.service_url }}`).

- The "Template repository URL" parameter with the `template_git_url` identifier (or another one if needed) of the Entities type: The URL of the template repository that the backend uses for cloning.
  In the additional settings, specify: Resource: "Templates", Relation: "Templates-Services", Field: "Property", Property: "URL".

Additional parameters depend on your template and must be configured according to the variables used in the template.

For the "Upgrade service from template" action, configure fields that collect the data the backend needs to create a merge request.
This data will be populated automatically from entities of the "Services" resource or from the related entity of the "Templates" resource:

- The "Variables used in templating" parameter with the `template_values` identifier (or another one if needed) of the String type: The variables that were used during the initial templating (they are stored in the entity property).
  Default value: `{{ .entity.properties.template_values }}`. It uses the saved variables from the entity property.
  If the parameter has a different name (for example, `template_vars`), use the corresponding identifier.

- The "Version" parameter with the `version` identifier (or another one if needed) of the Entities type: Gets the value of the new tag (version) from the template entity.
  In the additional settings, specify: Resource: "Templates", Relation: "Templates-Services", Field: "Property", Property: "Latest tag".

- The "Target repository" parameter with the `target_project_id` identifier (or another one if needed) of the String type: The internal ID of the service repository in GitLab in which the merge request is created.
  Default value: `{{ .entity.properties.project_id }}`. It uses the project ID from the entity property.
  If the ID property in the resource has a different name, use the corresponding identifier.

- The "Template repository" parameter with the `source_project_id` identifier (or another one if needed) of the Entities type: The internal ID of the template repository in GitLab.
  In the additional settings, specify: Resource: "Templates", Relation: "Templates-Services", Field: "Property", Property: "Repository ID".

- The "Target repository branch" parameter with the `target_repo_branch` identifier (or another one if needed) of the String type: The main branch of the service repository (for example, `master` or `main`) that the merge request targets.
  Default value: `{{ .entity.properties.default_branch }}`. It uses the default branch from the entity property.
  If the property identifier is different, use your own.

#### "Request configuration" tab

On the "Request configuration" tab, configure the action execution logic. The logic depends on the action being configured.

For the "Create GitLab project" action, fill in the following fields:

- "Action type": Select the built-in action type (the `BuiltIn` value).
- "Built-in action type": Select `CreateGitlabProject` to create a project in GitLab.
- "URL": Specify the URL of your GitLab instance (for example, `https://gitlab.example.com`).

Request body of the "Create GitLab project" action:

```yaml
initialize_with_readme: false
name: '{{ .name }}'
namespace_id: '{{ .group }}'
path: '{{ .path }}'
```

This request body passes the project name, group ID, and repository path to the GitLab API.
For details on the settings, see [CreateGitlabProject](../actions/gitlab/creategitlabproject/).

For the "Create repository from template" action, fill in the following fields:

- "Action type": Select the built-in action type (the `BuiltIn` value).

- "Built-in action type": Select `CreateRepositoryFromTemplate` to create a repository from a template.

- "URL": Specify the URL of your GitLab instance (for example, `https://gitlab.example.com`).

Request body of the "Create repository from template" action:

```yaml
sourceBranch: "master"
templateRepositoryUrl: "{{ .template_git_url }}"
targetRepositoryUrl: "{{ .service_git_url }}"
values:
  <KEY>: <VALUE>
```

Replace `<KEY>` and `<VALUE>` with the names and values of your template variables.
Add a separate line for each variable.

In the request body:

- `sourceBranch`: The branch in the template repository from which the code will be cloned (usually `master` or `main`).

- `templateRepositoryUrl`: The template repository URL from the action parameter.

- `targetRepositoryUrl`: The URL of the target service repository.

- `values`: Key-value pairs used to replace variables in the template files.
  The format and set of these variables depend entirely on how your template is configured.

For details on the settings, see [CreateRepositoryFromTemplate](../actions/gitlab/createrepositoryfromtemplate/).

For the "Upgrade service from template" action, fill in the following fields:

- "Action type": Select the built-in action type (the `BuiltIn` value).

- "Built-in action type": Select `CreateGitlabMergeRequest` to create a merge request in GitLab.

- "URL": Specify the URL of your GitLab instance (for example, `https://gitlab.example.com`).

Request body of the "Upgrade service from template" action:

```yaml
merge_request_spec:
  source_branch: update-to-{{ .version }}
  target_branch: {{ .target_repo_branch }}
  title: update-to-{{ .version }}
source_project_tag: {{ .version }}
source_project_id: "{{ .source_project_id }}"
target_project_id: "{{ .target_project_id }}"
values: {{ .template_values | nindent 4 }}
```

In the request body:

- `source_branch`: A new branch is created with a name that includes the tag version.

- `target_branch`: The target branch of the service that the merge request targets.

- `title`: The merge request title, including the version.

- `source_project_tag`: The template version tag from which the changes are taken.

- `source_project_id` and `target_project_id`: The IDs of the template and service repositories.

- `values`: The templating variables, applied again to render the changes correctly.

For details on the settings, see [CreateGitlabMergeRequest](../actions/gitlab/creategitlabmergerequest/).

#### "Update" tab

For the "Create GitLab project" and "Create repository from template" actions, enable the "Entity property updates" option and configure update rules that write information to the entity for which the action was run.
The entity update rules depend on the action being configured.

For the "Create GitLab project" action, use the identifiers of the corresponding resource properties:

- The "Repository ID" property: Use the `{{ .response.id }}` source. It saves the ID of the created project from the GitLab API response.

- The "URL" property: Use the `{{ .response.web_url }}` source. It saves the URL of the created project.

- The "Default branch" property: Use the `{{ .response.default_branch }}` source. It saves the default branch name.

The source identifiers (`id`, `web_url`, `default_branch`) are taken from the [GitLab Projects API](https://docs.gitlab.com/api/projects/#get-a-single-project) specification.

For the "Create repository from template" action, use the identifier of the corresponding resource property.
For the "Variables used for templating" property, use the `{{ toYAML .response.values.merged }}` source.
This expression saves the variables in YAML format.
These variables are critical for future service upgrades from the template because they must be applied again.
The data is received as a response to a successful action run.

For the "Create repository from template" action, also enable the "Creating relations for the entity" option so that the action automatically creates a relation between the created service and its template:

- "Resource": Select "Services".

- "Relation": Select the "Templates-Services" relation.

- "Parent entity slug": Use `{{ .property.template_ddp_slug }}`, the template identifier from the action parameter.

  {{< alert level="warning" >}}
  If the parameter identifier is not `template_ddp_slug`, replace it with the correct one.
  {{< /alert >}}

- "Child entity slug": Leave it empty. The current entity for which the action runs will be used.

#### "Authorization" tab

On the "Authorization" tab, configure the credentials for running the action. The settings depend on the action.

For the "Create GitLab project" action, configure the credentials.
In the "Identifier" field, specify `token`, and in the "Credentials" field, select the credential that contains a GitLab token with the permissions required to create projects in the specified group.

For the "Create repository from template" action, configure credentials with the `password` and `username` identifiers.
For `password`, select the credential that contains a GitLab password (token) with the required repository access permissions; for `username`, select the credential that contains the GitLab username.

For the "Update service from template" action, configure credentials with the `password` and `username` identifiers.
For `password`, select the credential that contains a GitLab password (token) with the required repository access permissions; for `username`, select the credential that contains the GitLab username.

## Process

You must have a process that combines the actions for creating a project in GitLab and creating a service from a template into a single sequential automated chain.
The process must run the creation of a project in GitLab and then the creation of a repository from the template, in sequence.
If the process does not exist yet, [create it](#creating-a-process).

Check that on the "Main information" tab, the "Services" resource is selected in the "Resource" field.

On the "Configuration" tab, the actions must be in the following order:

1. Start.
1. Create GitLab project.
1. Create repository from template.
1. End.

### Creating a process

To create a process, go to "Self-service" → "Processes".
Create a new process.
To do this, click "Create" in the upper right corner.

A window for creating a new process opens, with several tabs.

#### "Main information" tab

Fill in the following fields:

- "Name": `Create service from template`.

- "Resource": Select the "Services" resource.

- "Owner": [Specify the owner].

- "Owning team": [Specify the owner team].

#### "Configuration" tab

On the "Configuration" tab, configure the process steps and their execution order.
The task and action names are examples.
Add the following elements:

1. Add a "Task" element named "Create GitLab project" and select the "Create GitLab project" action to run.
1. Add a "Task" element named "Create repository from template" and select the "Create repository from template" action to run.
1. Add an "End" element to complete the process.

After adding all elements, configure the execution order:

1. Start.
1. Create GitLab project.
1. Create repository from template.
1. End.

## Automation

You must have an automation that runs the action for creating a merge request to upgrade services when a new template version appears.
The automation tracks changes to the "Latest tag" property in entities of the "Templates" resource and, when a new tag is detected, automatically runs the "Upgrade service from template" action for all related services.
If the automation does not exist yet, [create it](#creating-an-automation).

### Checking the automation

On the "Configuration" tab, check the trigger settings.
The "Resource" field must be set to "Templates", and the "Condition" field must contain `{{ ne .entity.properties.repository_last_tag "v1.0.0" }}` or a similar condition if your check is configured differently.

This condition fires every time the property with the `repository_last_tag` identifier in an entity of the "Templates" resource is not equal to the specified value (`v1.0.0`).
The condition is checked only once when the `repository_last_tag` parameter changes.
If the "Latest tag" property has a different identifier (for example, `last_tag`), use it in the condition: `{{ ne .entity.properties.last_tag "v1.0.0" }}`.

Check the execution settings.
The "Execution type" field must be set to "Actions", and the "Execution object" field must be set to the "Upgrade service from template" action.

Check that the "Run execution for related resources" parameter is enabled and the resource relations are configured.
The "Resource" field must be set to "Templates", and the "Relation" field must be set to the "Templates-Services" relation.

### Creating an automation

To create an [automation](../automations), go to "Self-service" → "Automations".
Create an automation.
To do this, click "Create" in the upper right corner.

A window for creating a new automation opens, with several tabs.

#### "Main information" tab

Fill in the following fields:

- "Name": [Specify the automation name].

- "Owner": [Specify the owner].

- "Owning team": [Specify the owner team].

#### "Configuration" tab

On the "Configuration" tab, configure the automation run logic.

A trigger determines when the automation runs.
Add a new trigger.
In the "Resource" field, select "Templates", and in the "Condition" field, specify `{{ ne .entity.properties.repository_last_tag "v1.0.0" }}`.

{{< alert level="warning" >}}
The condition uses the identifier of the "Templates" resource property that stores the latest tag of the Git repository.
If your identifier is different, use the correct one.
{{< /alert >}}

This condition fires every time the `repository_last_tag` property in an entity of the "Templates" resource is not equal to the specified value (`v1.0.0`).
The condition is checked only once when the `repository_last_tag` parameter changes.

{{< alert level="info" >}}
If you plan to create several automations, each of which must respond to property changes in entities of the same resource, specify the condition in the `{{ and (ne .entity.properties.repository_last_tag "v1.0.0") (eq .entity.slug "example") }}` format to additionally filter which automation runs in each specific case.
Without the `(eq .entity.slug "example")` condition, each automation will respond to changes in every entity.
{{< /alert >}}

Configure the action execution.
In the "Execution type" field, select "Actions", and in the "Execution object" field, select the "Upgrade service from template" action.

The automation watches for changes in entities of the "Templates" resource, but the action upgrades entities of the "Services" resource that are related to the template in which the `repository_last_tag` property was updated.
Enable the "Run execution for related resources" parameter and configure the resource relations.
In the "Resource" field, select "Templates", and in the "Relation" field, select the "Templates-Services" relation.

When the tag is updated in the "Templates" resource, the system finds all entities of the "Services" resource related to this template and runs the "Upgrade service from template" action for each of them, creating a merge request with changes from the new template version.

## Creating a service

The portal is now configured to create services from a template and automatically upgrade them when a new template version is released.

To create a service:

1. Go to the "Catalog" tab and open the "Services" resource.
1. Click "Create" in the upper right corner.
1. In the window that opens, fill in the required fields on the "Main information" tab.
   If you configured everything according to this guide, you do not need to fill in data on the other tabs.
   The data will be filled in automatically later.
1. To the right of the new service record, click the blue button with three vertical dots and select "Run process".
1. In the drop-down list, select the `Create service from template` process.
1. If needed, fill in the template variables and run the process.
1. If everything is configured correctly, the service is created.
   When template upgrades are available, merge requests with the changes will be created for the service.
   To verify that the service was created correctly, open it and go to the "Actions" tab and check the "Execution history" table.
   All [actions](../actions/overview/) must have the `Success` status.
