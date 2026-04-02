---
title: Templating
---

The platform supports templating using [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax). Details and usage examples are described in the relevant sections of the documentation.

In addition to the standard functions, the following are available:

* Built-in platform functions.
* Functions from the [sprig](https://masterminds.github.io/sprig/) library.

> Functions from the [sprig](https://masterminds.github.io/sprig/) library are not supported in data sources.

## Built-in Functions

### extractPart

`extractPart` splits the input string `s` by the specified delimiter `delimiter` and returns the part at the specified index `index`. If the index is outside the bounds of the array, an error is returned.

Parameters:

* `s` is the input string to split.
* `delimiter` is the delimiter used to split the input string.
* `index` — the index of the part to return after splitting.

Returns:

* A string representing the part at the specified index after splitting the input string.
* If the index is out of range, an error is returned.

Example in a template:

```go
{{ extractPart "ddp/demo-service" "/" 1 }} // Output: "demo-service"
```

### extractLastPart

`extractLastPart` splits the input string `s` by the specified delimiter `delimiter` and returns the last part. If the string is empty or the delimiter is not found, the original string is returned.

Parameters:

* `s` — the input string to split.
* `delimiter` — the delimiter used to split the input string.

Returns:

* A string representing the last part after splitting the input string.

Example in a template:

```go
{{ extractLastPart "ddp/demo-service" "/" }} // Output: "demo-service"
```

### toJSON

`toJSON` serializes the given value into a JSON string. Accepts any data type. If serialization is not possible, an error is returned.

Parameters:

* `v` is the value to serialize to JSON.

Returns:

* A JSON string representing the input value.
* The error that occurred during serialization.

Example in a template:

```go
{{ toJSON . }} // Output: JSON representation of the object in context
```

### replaceChar

`replaceChar` replaces all occurrences of the specified character in the string with another specified character.

Parameters:

* `s` is the source string to replace.
* `oldChar` — the character to replace.
* `newChar` — the character to replace with.

Returns:

* The modified string, where all occurrences of `oldChar` are replaced with `newChar`.

Example in a template:

```go
{{ replaceChar "hello/world" "/" "-" }} // Output: "hello-world"
```

### filteredItems

`filteredItems` filters an array of elements (maps) by the requested key value. Returns an array of elements whose value at the specified key matches the target value.

Parameters:

* `data` — the data array to filter.
* `key` — the key against which the value will be checked.
* `value` — the target value to compare.

Returns:

* An array of elements whose value at the given key matches the target value.
* An error if a problem occurs during filtering.

Example in a template:

```go
{{ filteredItems .items "name" "Alice" }} // Output: [{"name": "Alice", "age": 30}]
```

### anyOf

`anyOf` checks whether at least one element of the array meets the comparison condition.

Parameters:

* `operator` — comparison operator (eq, ne, lt, le, gt, ge).
* `value` — target value for comparison.
* `key` — key by which the value for comparison will be retrieved.
* `data` — data array (maps).

Returns:

* `true` if at least one value from the data array meets the condition.

Example in a template:

```go
{{ anyOf "eq" "value" "key" .data }} // Output: true if at least one record satisfies the condition
```

### allOf

`allOf` checks whether all array elements meet the comparison condition.

Parameters:

* `operator` — comparison operator (eq, ne, lt, le, gt, ge).
* `value` — target value for comparison.
* `key` — key by which the value for comparison will be retrieved.
* `data` — data array (maps).

Returns:

* `true` only if all values ​​meet the condition.

Example in a template:

```go
{{ allOf "gt" 10 "age" .users }} // Output: true if all users are over 10 years old
```

### getFieldValue

`getFieldValue` gets the value of a field by key from a structure represented as a map.

Parameters:

* `items` — a data structure represented as a map.
* `key` — the key to get the value by.

Returns:

* The value obtained by the key.
* An error if the structure does not exist or the key is not found.

Example in a template:

```go
{{ getFieldValue .item "name" }} // Output: The value of the "name" field from the .item structure
```

### findValueInDictArray

`findValueInDictArray` searches an array of dictionaries for an element where the value at the given key matches the specified value and returns the value of the other key.

Parameters:

* `data` — An array of dictionaries (map[string]interface{}) to search.
* `filterKey` — The key to filter by.
* `filterValue` — The value that the value at filterKey must match.
* `targetKey` — The key whose value should be retrieved from the found element.

Returns:

* The value corresponding to targetKey in the found dictionary.
* An error if the element is not found or the targetKey is missing.

Example in a template:

```go
{{ findValueInDictArray .items "environment" "test" "url" }} // Output: The value of the "url" key from the first dictionary found where "environment" is "test".
```

### generatePassword

`generatePassword` generates a random password with the given parameters.

Parameters:

* `length` — password length (default: 16).
* `includeUppercase` — include uppercase letters A-Z (default: true).
* `includeLowercase` — include lowercase letters a-z (default: true).
* `includeNumbers` — include numbers 0-9 (default: true).

Returns:

* A string with the generated password.
* An error if a password cannot be generated with the given parameters.

Examples in the template:

```go
{{ generatePassword }} // Output: Random password of length 16 characters
{{ generatePassword 12 }} // Output: Random password of length 12 characters
{{ generatePassword 8 true true true }} // Output: Password of length 8 characters
{{ generatePassword 10 false true true }} // Output: Password of length 10 characters, containing only lowercase letters and numbers
```

### encodeUnicode

`encodeUnicode` converts a string into a sequence of Unicode escape sequences. Each character of the string is encoded in the format `\uXXXX`, where `XXXX` is the four-digit hexadecimal representation of the character's code point.

Parameters:

* The input string to encode into Unicode escape sequences.

Returns:

* A string where each character is represented as a `\uXXXX` escape sequence.

Example in a template:

```go
{{ encodeUnicode "Hello" }} // Output: "\u041f\u0440\u0438\u0432\u0435\u0442"
{{ encodeUnicode "Hello" }} // Output: "\u0048\u0065\u006c\u006c\u006f"
```

### decodeUnicode

`decodeUnicode` decodes a string containing Unicode escape sequences back into a regular string. The function processes escape sequences of the form `\uXXXX`, where `XXXX` is a four-digit hexadecimal number representing a Unicode code point.

Parameters:

* A string with Unicode escape sequences to decode.

Returns:

* The decoded string, with all Unicode escape sequences converted to characters.
* An error if decoding fails (for example, if the escape sequences are in an invalid format).

Example in a template:

```go
{{ decodeUnicode "\u041f\u0440\u0438\u0432\u0435\u0442" }} // Output: "Hello"
{{ decodeUnicode "\u0048\u0065\u006c\u006c\u006f" }} // Output: "Hello"
```

## Global Variables

Global variables are shared variables that can be reused in templating when running actions.

To retrieve the value of a global variable, specify a string like this in the corresponding fields:

```go
{{ .global.<slug>.<key> }}
```

where:

- `global` — indicates that global variables are being accessed.
- `slug` — the identifier of the global variable set.
- `key` — the key of the variable whose value needs to be substituted.

> Global variables are stored in the DDP database in cleartext; their value can be retrieved by users through the web interface. It is not recommended to place sensitive data in global variables.

### Configuring Global Variables

Global variables are configured in the "Self-Service" → "Global Variables" section.

The following naming rules apply:

- The name of the global variable set must not be empty.
- The global variable set identifier cannot be empty and must meet the following conditions:
- Contain the characters `a-z`, `A-Z`, numbers, or underscores.
- Do not start with a number.
- The key of each variable in the set must not be empty and must meet the following conditions:
- Contain the characters `a-z`, `A-Z`, numbers, or underscores.
- Do not start with a number.
- The value of each variable in the set must not be empty.

## Team Variables

Team variables can be used in all actions, scripts, and processes.

Team variables are configured in the "Administration" → "Teams" section of the team editing menu.

Each user can edit variables for the teams they are a part of; this can be done in the user profile.

To retrieve the value of a team variable, use the following construct:

```go
{{ .team.<variable_name> }}
```

When running an action, the user must select a command whose variables will be substituted.

When running a script or process, the command is selected once, and its variables are used in all actions within it.

## Action Variables

### Action Parameters

Action parameters are accessible through the `{{ .property.* }}` context and contain the values ​​passed when running the action.

To retrieve the value of an action parameter, use the following construct:

```go
{{ .property.<property_slug> }}
```

where:

- `property` — indicates that action parameters are being accessed.
- `property_slug` — the ID of the parameter whose value needs to be substituted.

Parameter IDs can be viewed in the `Custom Form` tab of the action configuration window.

Usage examples:

```go
{{ .property.environment }} // Value of the "environment" parameter
{{ .property.count }} // Value of the "count" parameter
{{ .property.url }} // Value of the "url" parameter
```

### Action Response

The action response is available through the `{{ .response.* }}` context and contains the data returned after the action is executed.

To retrieve a value from an action response, use the following construct:

```go
{{ .response.<field_name> }}
```

where:

- `response` — indicates that the action response is being accessed.
- `field_name` — the name of the field in the response whose value should be substituted.

You can view the format of the response returned by an action either in the documentation describing the action or in the action entry in the DDP interface:

1. Open the action menu (the button with three dots in the action card).
1. Select 'Action Triggers'.
1. Open the configuration of one of the action triggers.
1. Find the **Response** column in the table.

Use examples:

```go
{{ .response.status }} // Response status
{{ .response.data.id }} // ID from response data
{{ .response.headers.auth }} // Authorization header value
```

> **Warning.** The `{{ .response.* }}` context can only be used in fields that describe rules for updating an entity after triggering an action.

## Credentials

Credentials are available in all actions, scripts, processes, widgets, data sources, and external services through the `{{ .credentials.* }}` context.

To retrieve the credential value, use the following construct:

```go
{{ .credentials.<credentials_slug> }}
```

where:

- `credentials` — indicates that credentials are being accessed.
- `credentials_slug` — the credential identifier whose value must be substituted.

In this case, the credential identifiers are those specified in the "Authorization" tab, in the "Credentials" section, in the DDP object configuration dialogs.

Usage examples:

```go
{{ .credentials.token }} // Access token
{{ .credentials.username }} // Username
{{ .credentials.password }} // Password
{{ .credentials.accessKeyId }} // Access Key ID for S3
{{ .credentials.secretAccessKey }} // Secret Access Key for S3
{{ .credentials.apiKey }} // API key
{{ .credentials.bearerToken }} // Bearer token
```

## Entity

The entity is accessible in widgets with the `Resource` scope, actions, processes, and scenarios through the `{{ .entity.* }}` context.

To retrieve the value of an entity field, use the following construct:

```go
{{ .entity.<field_name> }}
```

where:

- `entity` — indicates that an entity is being accessed.
- `field_name` — the name of the entity field whose value needs to be substituted.

### Primary Entity Fields

```go
{{ .entity.uuid }} // Entity UUID
{{ .entity.slug }} // Entity ID
{{ .entity.name }} // Entity Name
{{ .entity.description }} // Entity Description
``

### Entity Parameters

Entity parameters are accessible through the `{{ .entity.properties.* }}` context and contain user-defined parameters configured for a specific entity.

To retrieve the value of an entity parameter, use the following construct:

```go
{{ .entity.properties.<property_slug> }}
```

where:

- `entity` — indicates that an entity is being accessed.
- `properties` — specifies the entity parameters.
- `property_slug` — the identifier of the entity parameter whose value needs to be substituted.

Usage examples:

```go
{{ .entity.properties.projectId }} // Project ID from entity parameters
{{ .entity.properties.branch }} // Git branch from entity parameters
{{ .entity.properties.environment }} // Environment from entity parameters
{{ .entity.properties.apiUrl }} // API URL from entity parameters
{{ .entity.properties.version }} // Version from entity parameters
```

## Process Parameters

Each process can set general parameters, the values ​​of which can be used in all actions included in the script.

To obtain the value of a process parameter, use the following construct:

```go
{{ .process.<property_slug> }}
```

where:

- `process` — indicates that a process is being accessed.
- `property_slug` — the identifier of the process parameter whose value must be substituted.

The type and default value of parameters are specified in the process editing interface. The user can override the default value for each specific parameter when starting the process, if parameter editing is enabled.

Usage examples:

```go
{{ .process.deploymentUrl }} // Deployment URL from process parameters
{{ .process.branch }} // Git branch from process parameters
{{ .process.environment }} // Environment from process parameters
```

## Script Parameters

Each script can specify general parameters whose values ​​can be used in all actions within the script.

To get the value of a script parameter, use the following construct:

```go
{{ .workflow.<property_slug> }}
```

where:

- `workflow` — indicates that a script is being accessed.
- `property_slug` — the identifier of the script parameter whose value should be substituted.

The type and default value of parameters are specified in the script editing interface. The user can override the default value for each specific parameter when running the script if editing is enabled.

Usage examples:

```go
{{ .workflow.apiEndpoint }} // API endpoint from script parameters
{{ .workflow.notificationEmail }} // Notification email from script parameters
{{ .workflow.retryAttempts }} // Number of retry attempts from script parameters
```

## Process Storage

Storage is available only in processes and is used to pass data between actions. Action settings specify rules for writing to the storage (see ["Writing to Process Storage"](../../admin/actions/overview/#writing-to-process-storage)), and subsequent actions use placeholders to read data.

To retrieve a value from the storage, use the following construct:

```go
{{ .store.<key> }}
```

where:

- `store` — indicates that the process storage is being accessed. - `<key>` — the name of the key in the storage under which the value was saved (the **Key in Storage** field in the write rules).

Usage considerations:

- The storage is only available in processes; the `{{ .store.* }}` placeholders do not work in regular actions and scenarios.
- If the key is not in the storage (the action has not yet been executed, does not have write rules, or the write did not occur), the action will fail.
- If multiple actions are written for the same key, the last written value remains.

Usage examples:

```go
{{ .store.projectId }} // Project ID from the storage
{{ .store.orderRef }} // Order reference from the storage
```
