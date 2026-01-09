# Deckhouse Development Platform documentation

This is the source for the Deckhouse Development Platform documentation website.  

The project uses [Hugo](gohugo.io) SSG and the [hugo-web-product-module](https://github.com/deckhouse/hugo-web-product-module/) module for a theme.

Read [`hugo-web-product-module` README.md](https://github.com/deckhouse/hugo-web-product-module/blob/main/README.md) for information about content markup and other details.

## How to run the documentation site locally

To run locally:
1. Install werf and docker.
1. Run:

   ```bash
   make up
   ```

1. Open `http://localhost/products/development-platform/documentation/` in your browser (for the english version) or `http://ru.localhost/products/development-platform/documentation/` (for the russian version).
