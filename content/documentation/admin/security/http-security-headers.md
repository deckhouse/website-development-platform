---
title: HTTP security headers
weight: 50
---

Deckhouse Development Portal (DDP) supports the following HTTP headers to improve interaction security:

1. `Content-Security-Policy`
   - Configurable content security policy.
   - For a detailed description, see [Content Security Policy](#content-security-policy).

1. `X-Frame-Options`
   - Protection against clickjacking attacks.
   - Default value: `SAMEORIGIN`.
   - Configured in the `security.headers.xFrameOptions` module parameter.
   - Possible values: `DENY`, `SAMEORIGIN`.

1. `X-Content-Type-Options`
   - Prevention of MIME sniffing.
   - Value: `nosniff`.

1. `X-XSS-Protection`
   - Protection against XSS attacks (for legacy browsers).
   - Value: `1; mode=block`.

1. `Referrer-Policy`
   - Control of referrer information.
   - Value: `strict-origin-when-cross-origin`.

1. `Permissions-Policy` (formerly Feature-Policy)
   - Restriction of browser features.
   - Value in DDP Frontend responses: `geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), accelerometer=()`.
   - Value in DDP Backend responses: `geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), accelerometer=(), autoplay=(), encrypted-media=()`.

1. `Strict-Transport-Security` (HSTS)
   - Enforcement of HTTPS.
   - Value: `max-age=31536000; includeSubDomains; preload`.
   - DDP Backend adds the header only if the TLS connection is terminated on DDP Backend itself.
     DDP Frontend does not add this header.

{{< alert level="info" >}}
In a standard installation, the TLS connection is terminated on the Ingress controller, so DDP does not add the `Strict-Transport-Security` header.
To enable HSTS, configure it on the Ingress controller, for example, in the [`hsts`](/modules/ingress-nginx/cr.html#ingressnginxcontroller-v1-spec-hsts) parameter of IngressNginxController.
{{< /alert >}}

## Configuration

The headers are configured in the `security.headers` parameter of the `development-platform` module: in the `development-platform` ModuleConfig (`spec.settings.security.headers`) or in the module settings in the Deckhouse web UI.

Example of a ModuleConfig fragment with default values:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  settings:
    security:
      headers:
        # Enable or disable all security headers.
        enabled: true
        csp:
          # Allow loading images from any source.
          allowExternalImages: false
          # Allow embedding the portal in an iframe on any site and loading any site in a portal iframe.
          allowIframe: false
          # Space-separated additional sources added to the img-src, connect-src, frame-src, frame-ancestors CSP directives.
          # For example: https://example.com.
          additionalSources: ""
        # Value of the X-Frame-Options header (DENY or SAMEORIGIN).
        xFrameOptions: "SAMEORIGIN"
```

The `enabled` parameter enables or disables all security headers.
When set to `true`, the headers are added to HTTP responses; when set to `false`, they are not added.

## Applying headers

DDP Frontend adds the headers at the web server level when serving static files and HTML pages.
DDP Backend adds the headers through middleware to all API responses.

{{< alert level="info" >}}
All security headers are enabled by default (`enabled: true`).
If necessary, you can disable or configure them in the `security.headers` module parameter.
{{< /alert >}}

## Content Security Policy

Content Security Policy (CSP) is a security mechanism that helps prevent XSS attacks and other code injections by restricting the sources from which resources (scripts, styles, images, and others) can be loaded.

### Default CSP

DDP Frontend and DDP Backend send different CSP policies.

Default DDP Frontend policy:

```text
default-src 'self';
script-src 'self';
style-src 'self' 'unsafe-inline';
font-src 'self' data:;
img-src 'self' data:;
connect-src 'self' wss: ws:;
frame-src 'self';
worker-src 'self' blob:;
object-src 'none';
base-uri 'self';
form-action 'self';
frame-ancestors 'self';
upgrade-insecure-requests;
```

Default DDP Backend policy:

```text
default-src 'self';
script-src 'none';
style-src 'none';
img-src 'none';
font-src 'none';
connect-src 'self';
frame-src 'none';
frame-ancestors 'none';
object-src 'none';
base-uri 'self';
form-action 'self';
upgrade-insecure-requests
```

### Changing CSP

The `security.headers.csp` parameters change the policy as follows:

- `allowExternalImages: true`: Allows images from any source. In the DDP Frontend policy, the `img-src` directive is set to `'self' data: *`; in the DDP Backend policy, it is set to `*`.
  This is used, for example, for DDP object icons from external sources.
- `allowIframe: true`: Sets the `frame-src` and `frame-ancestors` directives to `*` in both policies.
  This is used, for example, for iframe widgets.
- `additionalSources`: Adds the listed sources to the `img-src`, `connect-src`, `frame-src`, and `frame-ancestors` directives in both policies.

{{< alert level="warning" >}}
With `allowIframe: true`, the portal can be embedded in an iframe on any site, and any site can be opened in portal iframe widgets.
The `X-Frame-Options` header is still sent, but modern browsers ignore it when the `frame-ancestors` directive is present.
This weakens protection against clickjacking attacks.

To allow only trusted sites, keep `allowIframe: false` and list them in the `additionalSources` parameter.
{{< /alert >}}
