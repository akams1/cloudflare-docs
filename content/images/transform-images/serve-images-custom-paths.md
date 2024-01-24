---
pcx_content_type: reference
title: Serve images from custom paths
weight: 27
meta:
    title: Serve images from custom paths
---

# Serve images from custom paths

You can use Transform Rules to rewrite URLs for every image that you transform through Images.

This page covers examples for the following scenarios:

- Serve images from custom paths
- Modify existing URLs to be compatible with transformations in Images
- Transform every image requested on your zone with Images

To create a rule, log in to the Cloudflare dashboard and select your account and website. Then, select **Rules** > **Transform Rules**.

## Before you start

Every rule runs before and after the transformation request.

If the path for the request matches the path where the original images are stored on your server, this may cause the request to fetch the original image to loop.

To direct the request to the origin server, you can check for the string `image-resizing` in the `Via` header:

`...and (not (any(http.request.headers["via"][*] contains "image-resizing")))`

## Serve images from custom paths

By default, requests to transform images through Images are served from the `/cdn-cgi/image/` path.
You can use Transform Rules to rewrite URLs.

### Basic version

Free and Pro plans only support string matching rules that do not require regular expressions.

This example lets you rewrite a request from `example.com/images` to `example.com/cdn-cgi/image/`:

```txt
---
header: Text in Expression Editor
---
(starts_with(http.request.uri.path, "/images")) and (not (any(http.request.headers["via"][*] contains "image-resizing")))
```

```txt
---
header: Text in Path > Rewrite to... > Dynamic
---
concat("/cdn-cgi/images", substring(http.request.uri.path, 7))
```

### Advanced version

{{<Aside type="note">}}

This feature requires a Business or Enterprise plan to enable regex in Transform Rules. Refer to [Cloudflare Transform Rules Availability](/rules/transform/#availability) for more information.

{{</Aside>}}

There is an advanced version of Transform Rules supporting regular expressions.

This example lets you rewrite a request from `example.com/images` to `example.com/cdn-cgi/image/`:

```txt
---
header: Text in Expression Editor
---
(http.request.uri.path matches "^/images/.*$") and (not (any(http.request.headers["via"][*] contains "image-resizing")))
```

```txt
---
header: Text in Path > Rewrite to... > Dynamic
---
regex_replace(http.request.uri.path, "^/images/", "/cdn-cgi/image/")
```

## Modify existing URLS to be compatible with transformations in Images

{{<Aside type="note">}}

This feature requires a Business or Enterprise plan to enable regex in Transform Rules. Refer to [Cloudflare Transform Rules Availability](/rules/transform/#availability) for more information.

{{</Aside>}}

This example lets you rewrite your URL parameters to be compatible with Images:

```
---
header: Text in Expression Editor
---
(http.request.uri matches "^/(.*)\\?width=([0-9]+)&height=([0-9]+)$")
```

```txt
---
header: Text in Path > Rewrite to... > Dynamic
---
regex_replace(
  http.request.uri.path,
  "^/(.*)\\?width=([0-9]+)&height=([0-9]+)$",
  "/cdn-cgi/image/width=${2},height=${3}/${1}"
)
```

Leave the **Query** > **Rewrite to...** > *Static* field empty.

## Pass every image requested on your zone through Images

{{<Aside type="note">}}

This feature requires a Business or Enterprise plan to enable regex in Transform Rules. Refer to [Cloudflare Transform Rules Availability](/rules/transform/#availability) for more information.

{{</Aside>}}

This example lets you transform every image that is requested on your zone with the `format=auto` option:

```
---
header: Text in Expression Editor
---
(http.request.uri.path.extension matches "(jpg)|(jpeg)|(png)|(gif)") and (not (any(http.request.headers["via"][*] contains "image-resizing")))
```

```txt
---
header: Text in Path > Rewrite to... > Dynamic
---
regex_replace(http.request.uri.path, "/(.*)", "/cdn-cgi/image/format=auto/${1}")
```