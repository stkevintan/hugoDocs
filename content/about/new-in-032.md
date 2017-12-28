---
title: Hugo 0.32 HOWTO
description: About page bundles, image processing and more.
date: 2017-12-28
keywords: [ssg,static,performance,security]
menu:
  docs:
    parent: "about"
    weight: 10
weight: 10
sections_weight: 10
draft: false
aliases: []
toc: true
---


**This page described unreleased features. You need to compile the master branch in https://github.com/gohugoio/hugo to get this up and running. It will be released soon.**

{{% note %}}
This documentation belongs in other places in this documentation site, but is put here first ... to get something up and running fast.
{{% /note %}}


Also see this demo project:

* http://hugotest.bep.is/
* https://github.com/bep/hugotest (source)
* For a list of "breaking changes" (not much): https://github.com/gohugoio/hugo/issues/4172


## Page Resources

### Organize Your Content

{{< figure src="/images/hugo-content-bundles.png" title="Pages with image resources" >}}

The content folder above shows a mix of content pages (`md` (i.e. markdown) files) and image resources.

{{% note %}}
You can use any file type as a content resource as long as it is a MIME type recognized by Hugo (`json` files will, as one example, work fine). If you want to get exotic, you can define your [own media type](/templates/output-formats/#media-types).
{{% /note %}}

The 3 page bundles marked in red explained from top to bottom:

1. The home page with one bundled image (`1-logo.png`)
2. The blog section with two bundled images and two bundled pages (`content1.md`, `content2.md`). Note that the `_index.md` represents the URL for this section.
3. An article (`hugo-is-cool`) with a folder with some images and one bundled content file (`cats-info.md`). Note that the `index.md` represents the URL for this article.

The content files below `blog/posts` are just regular standalone pages.

{{% note %}}
Note that changes to any resource inside the `content` folder will trigger a reload when running in watch (aka server or live reload mode), it will even work with `--navigateToChanged`.
{{% /note %}}

#### Sort Order

* Pages are sorted according to standard Hugo page sorting rules.
* Images and other resources are sorted in lexicographical order.

### Handle Page Resources in Templates


#### List all Resources

```html
{{ range .Resources }}
<li><a href="{{ .RelPermalink }}">{{ .ResourceType | title }}</a></li>
{{ end }}
```

For an absolute URL, use `.Permalink`.

**Note:** The permalink will be relative to the content page, respecting permalink settings. Also, bundled pages will not have a value for `RelPermalink`.

#### List All Resources by Type

```html
{{ with .Resources.ByType "image" }}
{{ end }}

```

Type here is `page` for pages, else the main type in the MIME type, so `image`, `json` etc.

#### Get a Specific Resource

```html
{{ $logo := .Resources.GetByPrefix "logo" }}
{{ with $logo }}
{{ end }}
```

#### Include Page Resource Content

```html
{{ with .Resources.ByType "page" }}
{{ range . }}
<h3>{{ .Title }}</h3>
{{ .Content }}
{{ end }}
{{ end }}

```


## Image Processing

The `image` resource implements the methods `Resize`, `Fit` and `Fill`:

Resize
: Resize to the given dimensin, `{{ $logo.Resize "200x" }}` will resize to 200 pixels wide and preserve the aspect ratio. Use `{{ $logo.Resize "200x100" }}` to control both height and width.

Fit
: Scale down the image to fit the given dimensions, e.g. `{{ $logo.Fit "200x100" }}` will fit the image inside a box that is 200 pixels wide and 100 pixels high.

Fill
: Resize and crop the image given dimensions, e.g. `{{ $logo.Fill "200x100" }}` will resize and crop to width 200 and height 100.


### Image Processing Options

In addition to the dimensions (e.g. `200x100`) where either height or width can be omitted, Hugo supports a set of additional image options:

Anchor
: Only relevant for `Fill`. This is useful for thumbnail generation where the main motive is located in, say, the left corner. Valid are `Center`, `TopLeft`, `Top`, `TopRight`, `Left`, `Right`, `BottomLeft`, `Bottom`, `BottomRight`. Example: `{{ $logo.Fill "200x100 BottomLeft" }}`

JPEG Quality
: Only relevant for JPEG images, values 1 to 100 inclusive, higher is better. Default is 75. `{{ $logo.Resize "200x q50" }}`

Rotate
: Rotates an image by the given angle counter-clockwise. The rotation will be performed first to get the dimensions correct. `{{ $logo.Resize "200x r90" }}`

Resample Filter
: Filter used in resizing. Default is `Box`, a simple and fast resampling filter appropriate for downscaling. See https://github.com/disintegration/imaging for more. If you want to trade quality for faster processing, this may be a option to test. 



### Performance

Processed images are stored below `<project-dir>/resources` (can be set with `resourceDir` config setting). This folder is deliberately placed in the project, as it is recommended to check these into source control as part of the project. These images are not "Hugo fast" to generate, but once generated they can be reused.

If you change your image settings (e.g. size), remove or rename images etc., you will end up with unused images taking up space and cluttering your project. 

To clean up, run:

```bash
hugo --gc
```


{{% note %}}
**GC** is short for **Garbage Collection**.
{{% /note %}}


## Configuration

### Default Image Processing Config

The below values from `config.toml` will be used if none is set in the template:

```toml
[imaging]
# Default resample filter used for resizing. Default is Box,
# a simple and fast averaging filter appropriate for downscaling.
# See https://github.com/disintegration/imaging
resampleFilter = "box"

# Defatult JPEG quality setting. Default is 75.
quality = 68
```





