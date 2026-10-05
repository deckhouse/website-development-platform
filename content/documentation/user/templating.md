---
title: Templating
description: Go templates in DDP, built-in and sprig functions, global and team variables, action, entity, process, and workflow contexts, process store.
---

Deckhouse Development Portal (portal) supports templating based on [Go templates](https://pkg.go.dev/text/template): expressions in curly braces are substituted into configuration fields where this is supported (actions, widgets, data sources, etc.). Below are the built-in functions and context variables (`{{ .property.* }}`, `{{ .entity.* }}`, and others).

In addition to the standard functions, the following are available:

* Built-in portal functions.
* Functions from the [sprig](https://masterminds.github.io/sprig/) library.

{{< alert level="info" >}}
Functions from the [sprig](https://masterminds.github.io/sprig/) library are not supported in data sources. The `env`, `expandenv`, and `getHostByName` functions are unavailable in all portal templates.
{{< /alert >}}

## Built-in functions

### extractPart

`extractPart` splits the input string `s` by the specified delimiter `delimiter` and returns the part at the specified index `index`. If the index is out of bounds, an error is returned.

Parameters:

* `s` — the input string to split.
* `delimiter` — the delimiter used to split the input string.
* `index` — the index of the part to return after splitting.

Returns:

* A string representing the part at the specified index after splitting the input string.
* If the index is out of bounds, an error is returned.

Example in a template:

```go-text-template
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

```go-text-template
{{ extractLastPart "ddp/demo-service" "/" }} // Output: "demo-service"
```

### toJSON

`toJSON` serializes the given value into a JSON string. Accepts any data type. If serialization is not possible, an error is returned.

Parameters:

* `v` — the value to serialize to JSON.

Returns:

* A JSON string representing the input value.
* An error that occurred during serialization.

Example in a template:

```go-text-template
{{ toJSON . }} // Output: JSON representation of the object in the current context
```

### toYAML

`toYAML` serializes the given value into a YAML string. Accepts any data type. If serialization is not possible, an error is returned.

Parameters:

* `v` — the value to serialize to YAML.

Returns:

* A string with the YAML representation of the input value.
* An error that occurred during serialization.

Example in a template:

```go-text-template
{{ toYAML . }} // Output: YAML representation of the object in the current context
```

### fromYAML

`fromYAML` parses a YAML string into a `map` for use in a template (for example, to merge with `dict` or access by key).

Parameters:

* `s` — a string containing YAML.

Returns:

* A `map` with the data from the string.
* An empty `map` if the string is empty.
* An error if the YAML is invalid.

Example in a template:

```go-text-template
{{ index (fromYAML "first: a\nsecond: b\n") "first" }} // Output: "a"
```

### replaceChar

`replaceChar` replaces all occurrences of a specified character in a string with another specified character.

Parameters:

* `s` — the source string in which to perform the replacement.
* `oldChar` — the character to replace.
* `newChar` — the character to replace it with.

Returns:

* The modified string, where all occurrences of `oldChar` have been replaced with `newChar`.

Example in a template:

```go-text-template
{{ replaceChar "hello/world" "/" "-" }} // Output: "hello-world"
```

### toSlug

`toSlug` converts an arbitrary string into a valid identifier form:

* The result contains only lowercase Latin letters and digits, with hyphens between fragments.
* The result has no leading hyphen.
* The result is no longer than 64 characters.
* Characters outside the allowed set are replaced with a single hyphen.
* Leading and trailing hyphens are removed.

Parameters:

* `s` — the source string.

Returns:

* A string in identifier form, or an empty string.

Example in a template:

```go-text-template
{{ toSlug "My Service/Name!" }} // Output: "my-service-name"
```

### filteredItems

`filteredItems` filters an array of items (maps) by a requested key value. Returns an array of items whose value at the specified key matches the target value.

Parameters:

* `data` — the array of data to filter.
* `key` — the key whose value is checked.
* `value` — the target value to compare against.

Returns:

* An array of items whose value at the specified key matches the target value.
* An error, if a problem occurs during filtering.

Example in a template:

```go-text-template
{{ filteredItems .items "name" "Alice" }} // Output: [{"name": "Alice", "age": 30}]
```

### anyOf

`anyOf` checks whether at least one element of the array satisfies the comparison condition.

Parameters:

* `operator` — the comparison operator (eq, ne, lt, le, gt, ge).
* `value` — the target value to compare against.
* `key` — the key whose value is used for comparison.
* `data` — the array of data (maps).

Returns:

* `true` if at least one value from the data array satisfies the condition.

Example in a template:

```go-text-template
{{ anyOf "eq" "value" "key" .data }} // Output: true if at least one record satisfies the condition
```

### allOf

`allOf` checks whether all elements of the array satisfy the comparison condition.

Parameters:

* `operator` — the comparison operator (eq, ne, lt, le, gt, ge).
* `value` — the target value to compare against.
* `key` — the key whose value is used for comparison.
* `data` — the array of data (maps).

Returns:

* `true` only if all values satisfy the condition.

Example in a template:

```go-text-template
{{ allOf "gt" 10 "age" .users }} // Output: true if all users are older than 10
```

### getFieldValue

`getFieldValue` retrieves a field's value by key from a structure represented as a map.

Parameters:

* `items` — a data structure in the form of a `map`.
* `key` — the key whose value should be retrieved.

Returns:

* The value obtained by the key.
* An error if the structure is empty or missing. If the key is not found, an empty value is returned without an error.

Example in a template:

```go-text-template
{{ getFieldValue .item "name" }} // Output: the value of the "name" field from the .item structure
```

### findValueInDictArray

`findValueInDictArray` searches an array of dictionaries for an item whose value at the specified key matches the given value, and returns the value of another key.

Parameters:

* `data` — the array of dictionaries (map[string]interface{}) to search.
* `filterKey` — the key used for filtering.
* `filterValue` — the value that the value at filterKey must match.
* `targetKey` — the key whose value should be retrieved from the found item.

Returns:

* The value corresponding to targetKey in the found dictionary.
* An error if the item is not found or targetKey is missing.

Example in a template:

```go-text-template
{{ findValueInDictArray .items "environment" "test" "url" }}  // Output: the value of the "url" key from the first found dictionary where "environment" equals "test".
```

### generatePassword

`generatePassword` generates a random password with the specified parameters.

Parameters:

* `length` — password length (default: 16).
* `includeUppercase` — include uppercase letters A-Z (default: true).
* `includeLowercase` — include lowercase letters a-z (default: true).
* `includeNumbers` — include digits 0-9 (default: true).

Returns:

* A string with the generated password.
* An error if a password cannot be generated with the specified parameters.

Examples in a template:

```go-text-template
{{ generatePassword }}                                   // Output: a random 16-character password
{{ generatePassword 12 }}                                // Output: a random 12-character password
{{ generatePassword 8 true true true }}                  // Output: an 8-character password
{{ generatePassword 10 false true true }}                // Output: a 10-character password using only lowercase letters and digits
```

### encodeUnicode

`encodeUnicode` converts a string into a sequence of Unicode escape sequences. Each character in the string is encoded in the format `\uXXXX`, where `XXXX` is the four-digit hexadecimal representation of the character's code point.

Parameters:

* `s` — the string to encode into Unicode escape sequences.

Returns:

* A string where each character is represented as a `\uXXXX` escape sequence.

Example in a template:

```go-text-template
{{ encodeUnicode "Привет" }} // Output: "\u041f\u0440\u0438\u0432\u0435\u0442"
{{ encodeUnicode "Hello" }}  // Output: "\u0048\u0065\u006c\u006c\u006f"
```

### decodeUnicode

`decodeUnicode` decodes a string containing Unicode escape sequences back into a regular string. The function handles escape sequences of the form `\uXXXX`, where `XXXX` is a four-digit hexadecimal number representing a Unicode code point.

Parameters:

* `s` — the string with Unicode escape sequences to decode.

Returns:

* The decoded string, where all Unicode escape sequences have been converted to characters.
* An error if decoding fails (for example, due to an incorrectly formatted escape sequence).

Example in a template:

```go-text-template
{{ decodeUnicode "\u041f\u0440\u0438\u0432\u0435\u0442" }} // Output: "Привет"
{{ decodeUnicode "\u0048\u0065\u006c\u006c\u006f" }}      // Output: "Hello"
```

### jwtSign

`jwtSign` creates a signed JWT from the given claims, signing key, and algorithm.

Parameters:

* `claims` — a `map` or a JSON string. Standard fields: "sub", "iss", "aud", "exp", "iat", "nbf".
* `signingKey` — the signing key: for HS256/HS384/HS512, a secret string; for RS256/ES256 and others, the PEM-encoded private key.
* `algorithm` — the signing algorithm: HS256, HS384, HS512, RS256, RS384, RS512, ES256, ES384, ES512, PS256, etc.
* `headers` — JWT headers: a `map` or a JSON string, for example "kid", "cty". The argument is required: to set no headers, pass an empty string `""` or an empty `map`.

Returns:

* A string with the signed JWT.
* An error for invalid parameters or key.

Examples in a template:

```go-text-template
{{ jwtSign (dict "sub" "user-123" "iss" "ddp") .credentials.secret "HS256" "" }}
```

## Global variables

Global variables are shared variables that can be reused in templating when actions are launched.

To substitute the value of a global variable, use the following construct in a field that supports templates:

```go-text-template
{{ .global.<SLUG>.<KEY> }}
```

where:

- `global` — indicates that a global variable is being accessed.
- `<SLUG>` — the identifier of the global variable set.
- `<KEY>` — the key of the variable whose value should be substituted.

{{< alert level="warning" >}}
Global variables are stored in the DDP database in plain text, and their values can be retrieved by users through the web interface. Do not store sensitive data in global variables.
{{< /alert >}}

### Configuring global variables

Global variables are configured in the "Self-service" → "Global variables" section.

The following naming rules apply:

- The name of a global variable set cannot be empty.
- The identifier of a global variable set cannot be empty and must meet the following conditions:
  - Contain only the characters `a-z`, `A-Z`, digits, or underscores.
  - Not start with a digit.
- The key of each variable in the set cannot be empty and must meet the following conditions:
  - Contain only the characters `a-z`, `A-Z`, digits, or underscores.
  - Not start with a digit.
- The value of each variable in the set cannot be empty.

## Team variables

Team variables can be used in all actions, workflows, and processes.

Team variables are configured in the "Administration" → "Teams" section, in the team editing menu, or in the user's profile. To edit team variables, a user must be a member of the team and have the global `update:team-variables` permission. For details, refer to the [role model](../../admin/security/rbac/#global-permissions).

To get the value of a team variable, use the following construct:

```go-text-template
{{ .team.<VARIABLE_NAME> }}
```

where `<VARIABLE_NAME>` is the name of the team variable.

When launching an action, the user must select the team whose variables will be substituted.
When launching a workflow or process, the team is selected once — its variables are used in all actions within it.

## Action variables

### Action parameters

Action parameters are available through the `{{ .property.* }}` context and contain the values passed when the action is launched.

To get the value of an action parameter, use the following construct:

```go-text-template
{{ .property.<PROPERTY_SLUG> }}
```

where:

- `property` — indicates that an action parameter is being accessed.
- `<PROPERTY_SLUG>` — the identifier of the parameter whose value should be substituted.

Parameter identifiers can be viewed on the "User form" tab in the action configuration window.

Usage examples:

```go-text-template
{{ .property.environment }}   // Value of the "environment" parameter
{{ .property.count }}         // Value of the "count" parameter
{{ .property.url }}           // Value of the "url" parameter
```

### Action response

The action response is available through the `{{ .response.* }}` context and contains the data returned after the action is executed.

To get a value from the action's response, use the following construct:

```go-text-template
{{ .response.<FIELD_NAME> }}
```

where:

- `response` — indicates that the action's response is being accessed.
- `<FIELD_NAME>` — the name of the field in the response whose value should be substituted.

For the response format, see the documentation for the specific action, or check it in the DDP interface:

1. Open the action's menu (the three-dot button on the action card).
1. Select "Action launch history".
1. Open the configuration of one of the action's runs.
1. Find the "Response" column in the table.

Usage examples:

```go-text-template
{{ .response.status }}        // Response status
{{ .response.data.id }}       // ID from the response data
{{ .response.headers.auth }}  // Value of the authorization header
```

{{< alert level="warning" >}}
The `{{ .response.* }}` context can only be used in fields that describe entity update rules after an action is run.
{{< /alert >}}

## Credentials

Credentials are available in all actions, workflows, processes, widgets, data sources, and external services through the `{{ .credentials.* }}` context.

To get the value of credentials, use the following construct:

```go-text-template
{{ .credentials.<CREDENTIALS_SLUG> }}
```

where:

- `credentials` — indicates that credentials are being accessed.
- `<CREDENTIALS_SLUG>` — the identifier of the credentials whose value should be substituted.

The `<CREDENTIALS_SLUG>` identifier matches the one specified on the "Authorization" tab, in the "Credentials" section, in the DDP object configuration dialogs.

Usage examples:

```go-text-template
{{ .credentials.token }}             // Access token
{{ .credentials.username }}          // Username
{{ .credentials.password }}          // Password
{{ .credentials.accessKeyId }}       // Access Key ID for S3
{{ .credentials.secretAccessKey }}   // Secret Access Key for S3
{{ .credentials.apiKey }}            // API key
{{ .credentials.bearerToken }}       // Bearer token
```

Access to `.credentials` works only if the expression is written directly in the body or configuration of an object. If a `{{ .credentials.* }}` expression comes in a substituted value, such as a global or team variable, a parameter, the launch context, or an action response, the portal removes it from the value.

## Entity

An entity is available in widgets with `Resource` scope, actions, processes, and workflows through the `{{ .entity.* }}` context.

To get the value of an entity field, use the following construct:

```go-text-template
{{ .entity.<FIELD_NAME> }}
```

where:

- `entity` — indicates that an entity is being accessed.
- `<FIELD_NAME>` — the name of the entity field whose value should be substituted.

### Basic entity fields

```go-text-template
{{ .entity.uuid }}           // Entity UUID
{{ .entity.slug }}           // Entity identifier
{{ .entity.name }}           // Entity name
{{ .entity.description }}    // Entity description
```

### Entity parameters

Entity parameters are available through the `{{ .entity.properties.* }}` context and contain the custom parameters configured for a specific entity.

To get the value of an entity parameter, use the following construct:

```go-text-template
{{ .entity.properties.<PROPERTY_SLUG> }}
```

where:

- `entity` — indicates that an entity is being accessed.
- `properties` — indicates the entity's parameters.
- `<PROPERTY_SLUG>` — the identifier of the entity parameter whose value should be substituted.

Usage examples:

```go-text-template
{{ .entity.properties.projectId }}     // Project ID from the entity's parameters
{{ .entity.properties.branch }}        // Git branch from the entity's parameters
{{ .entity.properties.environment }}   // Environment from the entity's parameters
{{ .entity.properties.apiUrl }}        // API URL from the entity's parameters
{{ .entity.properties.version }}       // Version from the entity's parameters
```

## Process parameters

For each process, you can define shared parameters whose values can be used in all actions that are part of the process.

To get the value of a process parameter, use the following construct:

```go-text-template
{{ .process.<PROPERTY_SLUG> }}
```

where:

- `process` — indicates that the process is being accessed.
- `<PROPERTY_SLUG>` — the identifier of the process parameter whose value should be substituted.

The parameter type and default value are set in the process editing interface. For each specific parameter, the user can override the default value when launching the process, if editing the parameter is allowed.

Usage examples:

```go-text-template
{{ .process.deploymentUrl }}    // Deployment URL from the process parameters
{{ .process.branch }}           // Git branch from the process parameters
{{ .process.environment }}      // Environment from the process parameters
```

## Process launch context

The launch context contains the data of the objects selected when a process is launched from the home page: system, microservice, environment, deployment, template. The context is available in default values of process parameters, in action bodies, in entity creation and update rules, in gateway conditions, and in loops.

To get a value from the context, use the following construct:

```go-text-template
{{ .context.<KEY> }}
```

where `<KEY>` is the path to the value in the context, for example `environment.slug`.

Entities are passed in the context with the `name`, `slug`, `uuid`, and `properties` fields. The `properties` keys are the identifiers of the entity parameters in the catalog.

Usage examples:

```go-text-template
{{ .context.environment.slug }}               // Identifier of the environment selected at launch
{{ .context.microservice.name }}              // Microservice name
{{ .context.environment.properties.domain }}  // The domain parameter of the environment
```

The behavior for a missing key depends on where the template is used:

- In process parameter values, a missing key is substituted as an empty string. If a required parameter is empty after substitution, the process does not start.
- In action bodies, gateway conditions, and loops, a missing key fails the step.
- If the context is empty, for example, when the process is launched from the catalog, a `{{ .context.* }}` expression in an action body is not substituted and remains in the text as is.

To make a process work when launched from the catalog too, set values from the context in process parameters and use the process parameters in actions: `{{ .process.<PROPERTY_SLUG> }}`.

## Workflow parameters

For each workflow, you can define shared parameters whose values can be used in all actions that are part of the workflow.

To get the value of a workflow parameter, use the following construct:

```go-text-template
{{ .workflow.<PROPERTY_SLUG> }}
```

where:

- `workflow` — indicates that the workflow is being accessed.
- `<PROPERTY_SLUG>` — the identifier of the workflow parameter whose value should be substituted.

The parameter type and default value are set in the workflow editing interface. For each specific parameter, the user can override the default value when launching the workflow, if editing the parameter is allowed.

Usage examples:

```go-text-template
{{ .workflow.apiEndpoint }}       // API endpoint from the workflow parameters
{{ .workflow.notificationEmail }} // Notification email from the workflow parameters
{{ .workflow.retryAttempts }}     // Number of retry attempts from the workflow parameters
```

## Process store

The store is only available in processes and is used to pass data between steps. Writing is done via rules in actions and the "Template" element (see [Process store](../../admin/processes/store/)); reading is done via placeholders in templates.

To read a value, use:

```go-text-template
{{ .store.<PATH> }}
```

where `<PATH>` is the path to the value in the store. It matches the "Target" field in the write rules. Nested keys are separated by dots:

```go-text-template
{{ .store.projectId }}
{{ .store.notification.engagements }}
```

The "Target" field of write rules supports array indices in square brackets, for example `items[0].status`. In placeholders, square brackets are not supported; use the `index` function to read an array item:

```go-text-template
{{ (index .store.items 0).status }}
```

### Process loop context

While a loop body inside a process is running, the `_loop` object is available in templates:

```go-text-template
{{ .store._loop.item }}    // current collection item or iteration number
{{ .store._loop.index }}   // 0-based index
{{ .store._loop.total }}   // total number of iterations
{{ .store._loop.first }}   // true on the first iteration
{{ .store._loop.last }}    // true on the last iteration
```

For nested loops, the parent context: `{{ .store._loop.parent }}`.
