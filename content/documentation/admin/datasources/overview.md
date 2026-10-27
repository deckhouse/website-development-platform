---
title: Overview
description: Datasources in Deckhouse Development Portal (DDP) — synchronization with external systems, rules, types, entity deletion, and logging.
weight: 10
---

Datasources are a Deckhouse Development Portal (DDP) mechanism that synchronizes information from external infrastructure systems (for example, GitLab or Kubernetes) and converts it into properties of entities.

Datasources also let you:

- Create new entities.
- Delete entities that were deleted from external infrastructure systems.
- Create relations between entities.

Each datasource is bound to a specific resource and, accordingly, can operate only on entities of that resource.
Relations, however, can be created between entities of the bound resource and any other entities.

## Datasource types

DDP supports two types of datasources.
Built-in (BuiltIn) datasources are datasources whose logic for polling infrastructure systems is shipped with DDP.
Webhooks (Webhooks) are a mechanism for receiving data in incoming POST requests from external systems. For details, see the ["Webhooks"](../webhooks/) section.

## Datasource parameters

Some datasource types support configurable parameters that let you further configure their behavior.
Details on the parameters of each datasource type are provided in the ["Datasource types"](./types/) section.

## Datasource rules

For each datasource, the system administrator configures rules that define how:

- New entities are created.
- New relations are created.
- Matching entities are found to update their properties.
- Entities to delete are found.

Rules are described in the `"source": "target"` format.
Rules use the [Go template](https://pkg.go.dev/text/template) syntax.

DDP supports 5 types of rules:

- Filter rules
- Creation rules (create rules)
- Matching rules (match rules)
- Update rules
- Relation creation rules (create relation rules)

### Filter rules

After data is received from an external system, filter rules are applied, and each subsequent rule processes the already filtered set of items.

In the datasource editing dialog, open the "Rules" tab.
In the "Filter rules" block, each rule has three fields:

1. Action: Keep only the items whose value of the selected field matches a regular expression (the "Keep only matches" option), or remove such items from the set (the "Remove matches" option).
1. Source: The path to a field in the data structure in the [Go template](https://pkg.go.dev/text/template) format.
1. Condition: A regular expression used to check the field value.

If there are no rules, all received items are passed further down the chain.
If no action is selected, the "Keep only matches" option is used.

To keep only records with a specific field value (for example, records whose `name` field value starts with `first`), select "Keep only matches".
In the source, specify an expression for this field, for example, `{{ .name }}`, and in the condition, specify a regular expression for matching values.
To skip records that match a pattern, select "Remove matches" and set the pattern in the condition.

{{< alert level="info" >}}
If the condition contains an invalid regular expression, the rule is not applied; the error is written to the synchronization log, and the datasource processing continues.
{{< /alert >}}

### Creation rules

Creation rules define how the information received from the datasource backend is used to create new entities.

For example, given the following specification of an external infrastructure system resource received through a datasource:

```yaml
# ...
metadata:
  name: example
  namespace: default
# ...
```

and the following creation rules:

```json
"create": [
  {
    "source": "{{ .metadata.name }}",
    "target": "Name"
  },
  {
    "source": "{{ .metadata.namespace }}-{{ .metadata.name }}",
    "target": "Slug"
  }
]
```

after the datasource is synchronized, an entity is created with the name (`Name`) set to `example` and the identifier (`Slug`) set to `default-example`.

### Matching rules

Matching rules define how to find existing entities, or entities just created by the datasource, to update their properties.

### Update rules

Update rules define which entity properties are populated automatically based on the specification of the corresponding resource from the external system.
The entity whose properties are populated is determined by the matching rules.

### Importing from a specification

Using the import mechanism, you can generate update rules and parameters of the updated resource automatically from a JSON, YAML, or OpenAPI data specification.

#### Configuring in the interface

To open the import from a specification, follow these steps:

1. On the "Main information" tab, select the resource the datasource will be bound to.
1. In the datasource editing dialog, open the "Rules" tab.
1. In the "Update rules" block, click "Import from specification".

#### Configuration steps

The import wizard consists of the following steps:

1. "Load data": Select the format (auto-detect, JSON, YAML, OpenAPI 3.x, Swagger 2.0) and paste the specification contents into the "Data" field.
   If the specification contains several schemas, you can select the schema for mapping in the "Select schema to import" field at the next step.
1. "Mapping configuration": The table shows the "Path in data", "Type in source", "Parameter slug", "Parameter name", and "Parameter type" columns.
   You can edit the parameter slug, name, and type, and select the rows to apply using checkboxes.
   Rows for which the resource already has a parameter with the same slug or name are marked as "Exists".
1. "Apply": Click "Apply" to add the generated rules to the "Update rules" block of the datasource.
   New resource parameters are created when the datasource is saved.

#### System behavior during import

You can enable the following options during import:

1. "Auto-create resource parameters": For specification fields that have no corresponding parameter in the resource yet, new parameters are suggested; they are added to the resource when the datasource is saved.
1. "Auto-fill update rules": A rule is created for each selected field: the source is set to a [Go template](https://pkg.go.dev/text/template) expression for the data path (for example, `{{ .metadata.name }}`), and the parameter is set to the resource parameter slug.
   After you click "Apply", these rules appear in the "Update rules" block on the "Rules" tab of the datasource.

#### Supported formats

Auto-detect, JSON, YAML, OpenAPI 3.x, Swagger 2.0.
Types from the specification (`string`, `integer`, `array`, `object`, and others) are mapped to DDP parameter types; if needed, you can change the type at the "Mapping configuration" step.

### Relation creation rules

Relation creation rules define which relations are created for the entity that matches the matching rules.
When you define relation creation rules, you specify a resource relation and a field from the specification of the external infrastructure system resource,
which is used to find the entity to create a relation with.

For example, given the following specification of an external infrastructure system resource received through a datasource:

```yaml
# ...
metadata:
  name: example
  namespace: default
# ...
```

and the following relation creation rule:

```json
"createRelations": [
  {
    "resourceRelationUuid": "<12345678-1234-5678-1234-567812345678>",
    "parentEntitySlug": "{{ .metadata.namespace }}",
    "childEntitySlug": ""
  }
]
```

the following applies:

- If the parent entity identifier (`parentEntitySlug`) is specified, the search is performed only among entities of the parent resource.
- If the child entity identifier (`childEntitySlug`) is specified, the search is performed only among entities of the child resource.
- Specify either the parent or the child identifier within a single rule.
- If both identifiers are set, the search is performed only in the parent resource.

If the parent resource has an entity with the `default` identifier, a relation is created between that entity and the entity of the resource the datasource is bound to that matches the matching rules.

## Synchronization

To configure synchronization, each datasource has the "Periodic synchronization" parameter, which enables or disables periodic synchronization,
and the "Launch frequency" parameter, a cron expression that defines the periodic synchronization interval.

When "Periodic synchronization" is enabled, the datasource automatically updates information from external infrastructure systems at the interval defined in the "Launch frequency" parameter.

You can also synchronize any datasource manually at any time in the web interface (the "Sync" item in the datasource menu) or through the API.

## Deleting entities

Entity deletion is configured with the "Delete non-existing entities" and "Delete only entities created by this data source" parameters.

When "Delete non-existing entities" is enabled, the datasource compares the list of resources received from the external infrastructure system with the list of existing entities of the resource.
Entities found in DDP that are missing from the received list are deleted from DDP.

When "Delete only entities created by this data source" is enabled, the datasource additionally checks how the entity to be deleted was created.
If the entity was created by this datasource, it is deleted; otherwise, the datasource moves on to the next entity in the list without deleting the current one.

The entity creator (origin) is determined by two fields in the entity specification:

```json
{
    "origin": {
        "type": "<datasource | manual | webhook | action>",
        "uuid": "<12345678-1234-5678-1234-567812345678>"
    }
}
```

## Run logging

For each datasource run, a record containing the full execution log is created in the database (DB).
The maximum number of such records is controlled by the "Maximum number of records" parameter in the datasource configuration.
When the maximum is reached, old records are deleted.

The `datasources.logging.enabled` module parameter lets you output datasource synchronization logs to stdout or hide them.
If set to `true`, logs are written to stdout; if set to `false` (default), they are not.
The parameter is set in the `development-platform` ModuleConfig (`spec.settings.datasources.logging.enabled`) or in the module settings in the Deckhouse web UI.

Example of a ModuleConfig fragment:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  settings:
    datasources:
      logging:
        enabled: true
```

{{< alert level="info" >}}
DB records with the full datasource run log are created regardless of the `datasources.logging.enabled` parameter value.
{{< /alert >}}

## System account

When configuring a datasource, select the system account used to run synchronization.

The datasource uses the details of this account to access the polled infrastructure system.

In addition to the system account, specify which of its details are used when running the datasource.
To do this, add new credentials with a specific identifier in the "Security / Credentials" section.

The value of this identifier can be substituted into the datasource configuration using the [Go template](https://pkg.go.dev/text/template) syntax.

For example, if a secret value with the `token` identifier is passed in a header for the datasource, you can specify it as follows:

```go-text-template
{{ .credentials.token }}
```
