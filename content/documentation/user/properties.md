---
title: Properties
---

For each resource, action, or workflow, the DDP administrator can add an unlimited number of properties of one of the following types:

* "Array" — a list of values;
* "Boolean" — a boolean value;
* "Date" — a date;
* "Entities" — an entity, one of whose properties can be selected as the value;
* "Enum" — an enumeration of values with a key and a displayed value;
* "JSON" — text in JSON format;
* "List" — a list of values with the ability to select one of them;
* "Markdown" — text in Markdown format;
* "Number" — a number;
* "Object" — an arbitrary object in JSON format;
* "Percentage" — a percentage;
* "String" — a string;
* "Teams" — teams;
* "URL" — a string in URL format;
* "Users" — users;
* "YAML" — text in YAML format.

## Resources

Once a property has been added for a resource, it is displayed for all entities of that resource, both in their cards and in the catalog table. A default value can be set for each property. The value of each property can be changed individually for each entity.

Whether properties are filled in is checked for each entity, and the check result is shown in the header of the entity's card.

### Synchronizing resource properties

For various reasons, entities may end up with properties that the resource no longer has. To remove such properties, use the "Sync properties" button in the resource menu.

When synchronizing properties:

* the identifier of each resource property is retrieved;
* the list of properties is retrieved for each entity of the resource;
* entity properties whose name does not match any of the resource's property identifiers are removed from the entity's specification.

## Actions and workflows

Each action and workflow has a user form consisting of properties that the user must fill in when launching it.

## Restrictions

The identifier of each property must:

* contain only the characters `a-z`, `A-Z`, digits, or underscores;
* not start with a digit.

## Configuration

* "Editable" — for each property, you can allow or disallow the user from editing it. If editing is disallowed, the user will not be able to change the property's value when launching actions or workflows, meaning the default value will always be used.
* "Required" — each property can be either required or optional. The value of a required property cannot be empty when launching actions or workflows. The value of an optional property, however, can remain empty without affecting the action's operation.
* "Hidden" — a hidden property is not displayed in entity tables and cards, or when launching actions or workflows.
* "Show in simplified form" — the property is displayed in the simplified form of the process launch window. Other properties keep their default values and are passed at launch. Required properties without a value are displayed in the simplified form regardless of this setting. The setting is available only for process properties.

## Property types

### Date

A property of type "Date" can hold a date value with a user-defined format. In the specification, the value is always stored in ISO 8601 format (`YYYY-MM-DDTHH:mm:ss.sssZ`).

#### Property configuration

* "Format" — configures how the date is displayed. If no format is explicitly set, the default format is used: `YYYY-MM-DDTHH:mm:ss.sssZ`. A description of the format configuration is available in the [Day.js documentation](https://day.js.org/docs/en/display/format). The format setting affects:
  * how the date is displayed in entity tables and cards;
  * the property's value when launching actions or workflows.
* "Current date by default" — substitutes the current date as the default value when editing entities, as well as when launching actions or workflows.
* "Default value" — a predefined default value. Not applied if the "Current date by default" toggle is enabled.

{{< alert level="info" >}}
When launching actions or workflows with the current date used as the default, the property must not be hidden in the user form. Otherwise, the current date will not be substituted.
{{< /alert >}}

When launching actions or workflows, values cannot be substituted into a "Date"-type property using [Go templates](https://pkg.go.dev/text/template).

### List

A property of type "List" allows selecting a single value from a predefined list.

### Enum

A property of type "Enum" allows selecting a single item from a predefined list. Unlike the "List" type, the "Enum" property uses a key-value pair, where the key is stored in the specification and the value is displayed to the user. This allows the displayed text to be changed without changing the keys.
