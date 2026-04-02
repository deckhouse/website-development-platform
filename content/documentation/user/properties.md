---
title: Parameters
---
For each resource, action, or scenario, the platform administrator can add an unlimited number of parameters of one of the following types:

* **Array** - list of values
* **Boolean** - Boolean value
* **Date** - date
* **JSON** - text in JSON format
* **Entities** - entity, one of whose parameters can be selected as a value
* **Enum** - enumeration of values ​​with a key and display value
* **List** - list of values ​​with the ability to select one of them
* **Markdown** - text in Markdown format
* **Number** - number
* **Object** - arbitrary object in JSON format
* **Percentage** - percentage
* **String** - string
* **YAML** - text in YAML format
* **Teams** - teams
* **URL** - string in URL format
* **Users** - users

## Resources

After adding a parameter to a resource, it will appear in all entities of that resource, both in their cards and in the catalog table. A default value can be set for each parameter. The value of each parameter can be changed individually for each entity.

Parameters are checked for completeness for each entity, and the result is displayed in the entity card header.

### Resource Parameter Synchronization

For various reasons, entities may have parameters that the resource does not have. To remove such parameters, use the "Synchronize Parameters" button in the resource menu.

When synchronizing parameters:

* Each resource parameter will receive its ID.
* A list of its parameters will be retrieved for each resource entity.
* Entity parameters whose names do not match any of the resource parameter IDs will be removed from the entity specification.

## Actions and Scenarios

For each action and scenario, a user form is defined, consisting of parameters that the user must fill in upon startup.

## Restrictions

The identifier of each parameter must:

* Contain the characters `a-z`, `A-Z`, numbers, or underscores
* Not start with a number

## Configuration

* **Editable Parameter** - Each parameter can be configured to allow or deny user editing. If editing is denied, the user will not be able to change the parameter value when running actions or scenarios; the default value will always be used.
* **Required Parameter** - Each parameter can be either mandatory or optional. The value of a mandatory parameter cannot be empty when running actions or scenarios. However, the value of an optional parameter can remain empty without affecting the functionality of the action.
* **Hidden Parameter** - A hidden parameter is not displayed in tables and entity cards, or when running actions and scenarios.

## Parameter Types

### Date

A "Date" type parameter can accept a date value with a user-defined format. In this case, the value is always stored in the ISO 8601 format (`YYYY-MM-DDTHH:mm:ss.sssZ`).

#### Parameter Configuration

* Format - Date display settings. If the format is not explicitly specified, the default format is used: `YYYY-MM-DDTHH:mm:ss.sssZ`. A description of the format configuration is available at [link](https://day.js.org/docs/en/display/format).
  * The format setting affects:
  * Date display in tables and entity cards;
  * Parameter value when running actions or scenarios.
* Current date by default - The current date is used as the default value when editing entities, as well as when running actions or scenarios.
* Default value - A predefined default value. Does not apply if the "Current date by default" toggle is enabled.

{{< alert level="info" >}}
When running actions or scripts and using the current date as the default, the parameter must not be hidden in the user form. Otherwise, the current date will not be populated.
{{< /alert >}}

When running actions or scripts, you cannot populate a date parameter with values ​​using the [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax).

### List

The "List" parameter allows you to select a single value from a predefined list.

### Enum

The "Enum" parameter allows you to select a single element from a predefined list. Unlike the "List" type, the "Enum" parameter uses a key-value pair, where the key is stored in the specification and the value is displayed to the user. This allows you to change the displayed text without changing the keys.
