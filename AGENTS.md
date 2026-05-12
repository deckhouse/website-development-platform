# AGENTS.md

This repository contains the Hugo source for the Deckhouse Development Platform documentation website.

Use this file as a quick entry point. Treat the detailed rules under `.cursor/rules/docs/` as normative when they apply.

## Repo Overview

- Main documentation content lives under `content/documentation/`.
- Hugo configuration lives under `config/_default/hugo.yaml`.
- Local run instructions live in `README.md`.
- The site uses the shared Hugo module `github.com/deckhouse/hugo-web-product-module`.

## Preferred Workflow

Work from the repository root.

```bash
make up
make build
make lint-markdown
make down
```

- Use `make up` to start the local docs site.
- Use `make build` to verify the site builds.
- Use `make lint-markdown` before handing off Markdown changes.
- Use `make lint-markdown-fix` only when automatic fixes are useful and safe.
- Use `make down` to stop local containers.

For local preview URLs, follow `README.md` as the source of truth:

- EN: `http://localhost/products/development-platform/documentation/`
- RU: `http://ru.localhost/products/development-platform/documentation/`

## Documentation Rules

When editing documentation pages:

- Do not use a first-level heading (`# ...`) in documentation files. Start at `##`.
- Always use fenced code blocks with an explicit language tag supported by Hugo/Chroma.
- Use Hugo shortcodes and alerts only as allowed by project rules.
- Keep YAML examples and front matter syntactically valid.
- Use meaningful link text. Avoid generic anchors such as "here" or "тут".
- In command examples, prefer `d8 k` instead of `kubectl`.

Primary references:

- `.cursor/rules/docs/global-style.mdc`
- `.cursor/rules/docs/hugo-supported-codeblock-languages.mdc`
- `.cursor/rules/docs/hugo-shortcodes.mdc`
- `.cursor/rules/docs/refs-editorial-policy-full.mdc`

## Language And Terminology

- Follow the repository glossary and terminology rules exactly.
- Do not replace approved terms with ad hoc synonyms.
- Do not write "платформа Deckhouse" when a specific product name is required.
- Use module names exactly as defined canonically, without humanizing or reformatting them.
- For Russian instructional text, use imperative "вы"-form.

Primary references:

- `.cursor/rules/docs/terminology.mdc`
- `.cursor/rules/docs/refs-glossary-full.mdc`

## EN/RU Parity

This repository is bilingual.

- Keep EN and RU pages semantically aligned when a paired page exists.
- Russian files must use the `.ru.md` suffix.
- When changing one language, check whether the paired page needs the same update.
- Localized media should follow the project naming pattern, for example `image.png` and `image.ru.png`.

Primary reference:

- `.cursor/rules/docs/ru-en-parity.mdc`

## Specialized Rules

If your change touches one of these areas, read the matching rule file before editing:

- CRD translation files: `.cursor/rules/docs/crd-translation-files.mdc`
- Front matter, links, and media conventions: `.cursor/rules/docs/frontmatter-links-media.mdc`
- OpenAPI `x-doc-*` fields: `.cursor/rules/docs/openapi-x-doc.mdc`

## Notes For Agents

- Keep this file short and practical. Do not duplicate the full policy files into new documents.
- Prefer small, consistent documentation edits over broad rewrites.
- If project instructions conflict, follow the more specific rule file, and treat the full editorial policy and glossary as authoritative.
