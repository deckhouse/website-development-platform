---
title: CreateRepositoryFromTemplate
weight: 10
---

{{< alert level="info" >}}
Running this action requires credentials:

- `password` — the password (token) of the user on whose behalf the action will be run.
- `username` — the username on whose behalf the action will be run.
{{< /alert >}}

CreateRepositoryFromTemplate — creates a new repository from a template in GitLab. The rendering mechanism is based on [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) and supports all built-in methods, as well as extensions added by the platform.

### Request example

```yaml
sourceBranch: main
sourceTag: v1.0.0
templateRepositoryUrl: https://gitlab.example.com/example-1.git
targetRepositoryUrl: https://gitlab.example.com/example-2.git
targetBranch: master
additionalIgnoreFiles:
  - .ignore
  - .example
values:
  key1: value1
  nested:
    enabled: true
    subkey: 123
```

### Request specification

| Name                       | Required | Description                                                                                                                            | Default value |
| -------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| templateRepositoryUrl      | Yes      | URL of the template repository                                                                                                              | -                       |
| targetRepositoryUrl        | Yes      | URL of the repository to be created as a result of running the action                                                                       | -                       |
| values                     | Yes      | Variables used during templating, in `key: value` format                                                                                    | -                       |
| additionalIgnoreFiles      | No       | List of files containing paths to exclude from the target repository. Populated similarly to [.templateignore](#templateignore)             | -                       |
| sourceTag                  | No       | Tag of the template repository to use during templating. If not specified, the template repository's branch is used                        | -                       |
| sourceBranch               | No       | Branch of the template repository to use during templating                                                                                  | main                    |
| targetBranch               | No       | Branch of the target repository to be created as a result of running the action                                                             | main                    |

### How it works

The platform:

1. Clones the template repository from the specified URL (`templateRepositoryUrl`), using `sourceTag`, `sourceBranch`, or the `main` branch as the ref, in that order of preference.
1. Reads the `values.yaml` file stored at the root of the repository and determines the default templating variables.
1. Reads the variables passed when the action is launched and merges them with the variables from `values.yaml`. Variables passed at launch take priority during the merge.
1. Reads the `.templateignore` file and determines the directories and files excluded from templating.
1. Renders the files from the templates, taking into account `values.yaml` and the variables passed to the action.
1. Changes the remote of the repository to the target one (`targetRepositoryUrl`) and pushes to the target branch (`targetBranch`) or to the `main` branch.

### Implementation details

The action supports templating of directory and file names. To do this, add an expression in [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) format to their name.
For example, the directory `src/{{ .module }}/utils`, given a `module` value of `example`, will be rendered as the directory `src/example/utils` in the target repository.

If, after rendering from the template, a file's content is empty, the file is not created. For example, a file with the following content:

```go
{{- if .createContent }}
- This is content that will be displayed if the createContent variable == true
{{- end }}
```

will not be created if the `createContent` variable is `false`. Similarly, files that are originally empty will not be created either.

If templating variables are missing from both the `values.yaml` file and the variables passed when the action is launched, rendering fails with an error and the target repository is not created.

### Template repository variables

To add default variables used during templating, create a `values.yaml` file with the appropriate content at the root of the repository.

Example `values.yaml` file:

```yaml
module: example
createContent: false
```

The `values.yaml` file is optional.

<a id="templateignore"></a>

### Excluding files

Some files may contain variables in [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) format that need to be preserved when rendering the repository from a template, for example Helm charts in the `helm` directory. The `.git` directory is always ignored.

To exclude such files from the rendering mechanism, add a `.templateignore` file with the appropriate content to the root of the repository.

Each line of `.templateignore` defines a single rule — a path or a mask. If the line contains `{{`, the entire line is executed as a single Go template with the same variables and functions used when substituting into file and directory names (for more information, see [How a rule line is processed](#templateignore-templates)). If the line does not contain `{{`, no substitution of variables from `values.yaml` or from the action's request is performed: the line is read as-is and compared with the relative path on disk; literals and the mask characters `*` and `**` are allowed.

Example of a `.templateignore` file containing only path masks, without substitution templates, to ignore the contents of the `helm` and `docs` directories:

```sh
helm/**
docs/**
```

#### Adding paths to ignore

1. At the root of the template repository, create or edit the `.templateignore` file (one rule per line).
1. Each line is a single rule: a path from the repository root. Regular path characters and masks are allowed: an asterisk `*` within a segment name, and the sequence `**` for arbitrary directory nesting. The platform matches the relative path against the mask according to built-in rules (similar to common conventions used for masks in ignore files in version control systems).
1. To substitute a path fragment from `values.yaml` or from the `values` field of the action's request, use Go template constructs (`{{ ... }}`) in the line. If the line contains `{{`, the platform processes the entire line from start to finish as a single Go template: you cannot leave part of the line as "plain text" and template only the middle of the path. See examples in [Go template examples in .templateignore](#templateignore-go-examples).
1. Empty lines and lines starting with `#` are skipped when parsing the file — you can use them for comments.

#### Examples without substitution (path masks only)

Individual files at the root:

```sh
package-lock.json
yarn.lock
LICENSE
.env.local
```

Entire directories and typical build artifacts:

```sh
vendor/**
node_modules/**
dist/**
build/tmp/**
```

Nesting with a mask:

```sh
docs/**/*.pdf
charts/*/values.schema.json
.github/workflows/**
```

Secrets by extension across the entire tree:

```sh
**/*.pem
**/*.key
```

<a id="templateignore-templates"></a>

#### How a rule line is processed

The variables available for expanding rules are the same ones merged from `values.yaml` at the root of the template repository and from the `values` field in the action's request (the `values` field in the request takes priority).

For each non-empty line from `.templateignore` or from a file listed in `additionalIgnoreFiles`, the platform does the following.

1. The line is always added to the rule list exactly as it appears in the file, unchanged. This is the line that is compared with the file path first — this is needed for cases where the on-disk names still contain fragments like `{{ .module }}` before renaming.
1. If the line contains `{{`, the platform runs the entire line once through the [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) engine, with the same capabilities used when substituting into file and directory names (built-in platform functions and the Sprig set). If the line does not contain `{{`, this step is skipped.
1. If the substitution step was performed and the resulting text differs from the line in the file (including the case where the result is empty), a second entry is added to the rule list — with this resulting text. As a result, a single line in the file can produce two entries in the list. When checking a file path, both are checked: a match with either one is enough for the file to be covered by the rule.

Then, while traversing the tree, for each file path, the path relative to the root of the cloned copy is calculated (with forward slashes). The path is compared against each rule: first using mask-matching rules (including `*` and `**`), and, if necessary, by an exact match between the rule string and the relative path.

This way, a single line in the file can define two rules: for example, the original `charts/{{ .name }}/**` (so as not to affect the path before directories are renamed), and the expanded `charts/billing/**` (to match the path after variable substitution into on-disk names). This is why the rules work both "before" and "after" the directory-renaming step of the template.

If the template in a line has a syntax error or references a missing field, rule expansion fails with an error, and repository rendering does not continue.

The `additionalIgnoreFiles` field in the action specifies the names of additional files at the root of the repository; each of them contains rule lines just like `.templateignore`, with the same template expansion. However, the meaning is different: matched paths are removed from the working copy (the directory or file is deleted), and this is done twice — at the beginning and at the end of the processing chain — so that these objects do not participate in further steps and do not end up in the resulting repository.

The `.templateignore` file, on the other hand, means "do not template": for matched paths, the platform does not substitute variables into file contents and does not apply template-based renaming to the corresponding directory paths. The files and directories themselves are not deleted just because of an entry in `.templateignore` — they remain in the copy, but are processed as plain text and plain names, without the templating step.

<a id="templateignore-go-examples"></a>

#### Go template examples in .templateignore

Each example below shows fragments of `values.yaml` (or the equivalent fields in the request's `values`) and lines from `.templateignore`. Several independent rules are defined by several lines in the file: the result of one template line is one masked line; line breaks within the template's result do not split the rule into several.

The directory name in the rule comes from `values.yaml` or from the action's `values`:

`values.yaml`:

```yaml
project: payment-gateway
```

`.templateignore`:

```go
{{ .project }}/legacy/**
```

The rule set will include the lines `{{ .project }}/legacy/**` and `payment-gateway/legacy/**`.

A path segment from a variable and a fixed part that is preserved after the variable's value is substituted:

`values.yaml`:

```yaml
lang: ru
```

`.templateignore`:

```go
apps/{{ .lang }}/messages.yaml
```

A conditional rule on a single line works as follows: when `skipGenerated: false`, the substitution produces an empty string, which is still added as a second rule. The original line with `{{` is used as a path mask and usually does not match any real paths. When `skipGenerated: true`, the second rule becomes `generated/**`:

`values.yaml`:

```yaml
skipGenerated: false
```

`.templateignore`:

```go
{{- if .skipGenerated }}generated/**{{- end }}
```

A variant for "ignore only non-production":

`values.yaml`:

```yaml
tier: staging
```

`.templateignore`:

```go
{{- if ne .tier "prod" }}mock/**{{- end }}
```

Using `printf` to build a mask string (convenient when the chart name is a variable):

`values.yaml`:

```yaml
chartName: wordpress
```

`.templateignore`:

```go
{{ printf "charts/%s/**" .chartName }}
```

A default value for an "empty" value — the `default` function from the Sprig set. The field must exist in the data (otherwise expansion will fail with `missingkey=error`); for "not set in YAML", add a key with an empty string, or use the `if` / `index` construct:

`values.yaml`:

```yaml
envName: ""
```

`.templateignore`:

```go
{{ default "dev" .envName }}/secrets/**
```

With an empty `envName`, the rule set will include both `{{ default "dev" .envName }}/secrets/**` and `dev/secrets/**`.

An optional path segment ([`with`](https://pkg.go.dev/text/template#hdr-Actions)):

`values.yaml`:

```yaml
analyticsModule: tracking
```

`.templateignore`:

```go
{{ with .analyticsModule }}{{ . }}/vendor/**{{ end }}
```

If `analyticsModule` is empty, the template produces an empty string (the expansion rules are described above).

Accessing a nested structure's field by a string key ([`index`](https://pkg.go.dev/text/template#hdr-Functions)):

`values.yaml`:

```yaml
regions:
  primary: eu-west
```

`.templateignore`:

```go
configs/{{ index .regions "primary" }}/bootstrap.yaml
```

Trimming whitespace from a name segment — the `trim` function from the Sprig set:

`values.yaml`:

```yaml
serviceName: " billing-api "
```

`.templateignore`:

```go
{{ trim .serviceName " " }}/logs/**
```

Several fragments in a single mask:

`values.yaml`:

```yaml
base: services
variant: canary
```

`.templateignore`:

```go
{{ .base }}/{{ .variant }}/**/*.tmp
```

#### Examples for additionalIgnoreFiles

The action's specification lists the names of files at the root of the repository (for example, `.ship-ignore`, `.ci-remove`). The line format inside such files is the same: `#` comments, empty lines, path masks, and, if needed, Go templates in lines.

The `.ship-ignore` file in the template:

```sh
# excluded from the target repository
local/fixtures/**
scratchpad.md
```

Fragment of the action's request:

```yaml
additionalIgnoreFiles:
  - .ship-ignore
```

A template in a file used with `additionalIgnoreFiles` (removing a directory, with the name coming from variables):

The `.env-drop` file at the root of the template:

```go
{{ .obsoleteDir }}/**
```

With `obsoleteDir: legacy-ui` from `values`, after expansion, the deletion list will contain both `{{ .obsoleteDir }}/**` and `legacy-ui/**` — matching paths will be removed from the copy before the final steps.

{{< alert level="info" >}}
Note the difference:
- `.templateignore` leaves files on disk but disables path renaming and content rendering for them;
- lists from `additionalIgnoreFiles` remove matched paths from the working copy.
{{< /alert >}}

### Example directory structure of a template repository

```sh
├── example-folder-01
│   ├── example-file-01
│   └── {{ .example }}-file-02
├── {{ .example }}-folder-02
│   └── ...
├── values.yaml
└── .templateignore
```

If the `example` variable is set to `new` when the repository is rendered, the resulting structure after rendering will look as follows:

```sh
├── example-folder-01
│   ├── example-file-01
│   └── new-file-02
├── new-folder-02
│   └── ...
├── values.yaml
└── .templateignore
```

<a id="local-debugging"></a>

### Local debugging

The `ddp-render-dir` utility is available for local debugging of templates.

The utility:

1. Creates a copy of the source directory.
1. Renders the files in this directory using the same rules as the action that creates repositories from templates.

Command-line flags for running it:

* `--source-dir` — the source directory to render.
* `--target-dir` — the directory where the rendering result will be placed.
* `--values` (optional) — path to a `values.yaml` file with variables to use during rendering.
* `--ignore-files` (optional) — a list of files containing paths to exclude from the target repository.
