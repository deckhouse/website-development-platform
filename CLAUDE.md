# Deckhouse Development Portal documentation site

Documentation of Deckhouse Development Portal (`productCode: development-platform`), built with Hugo on hugo-web-product-module.
Writing and markup rules come from the `deckhouse-ai-tools` plugins (`deckhouse-writing`, `deckhouse-product-site`),
enabled in `.claude/settings.json`. This file holds only facts about this product and repository.

## Product name

The product is called Deckhouse Development Portal.
The former names Deckhouse Development Platform and Express 42 Platform are obsolete; replace them in prose.
The repository name and `productCode` keep `development-platform`.

## Product source

The product source is the `ddp` repository (`git@fox.flant.com:deckhouse/ddp/development-platform.git`).
Check product behaviour in its handlers, seeds and locales; ADRs in `adrs/` are context only.
If the code and an ADR disagree, tell the user and follow the code.
Interface strings are in `images/frontend/src/locales/` (`en` and `ru`): form field names, hints and other UI text in the documentation must match them.

## Languages

Write new pages and release notes only in Russian (`.ru.md`). Add the English version only when the user asks for it.

## Release notes

- When you document a feature, also add it to the release notes of the latest version in `content/documentation/release-notes/`, unless the user says otherwise.
- A release notes item is a short description of the change in one or two lines with a link to the detailed documentation page.

## Repository notes

Besides the checks of the plugin, build the site into a scratch directory and check the `href` and `id` attributes of anchors on the changed pages
in `<scratchpad>/out/ru/documentation/**/index.html`. Use the Hugo image from `docker-compose.yml` as `<hugo-image>`:

```shell
docker run --rm -v "$PWD:/src:ro" -v <scratchpad>/out:/out -w /src <hugo-image> build -d /out --noBuildLock
```
