---
title: "Setting up Hugo with TailwindCSS"
date: 2025-04-07
layout: post
current: post
navigation: True
author: l4sh
category: development
tags:
  - hugo
  - web development
  - tailwindcss
  - static sites
comments: true
---

This is a quick guide on how to set up a Hugo site with TailwindCSS. To be fair this is more of a reminder for future self than anything else.

The process is pretty straightforward and the official documentation is really good. https://gohugo.io/functions/css/tailwindcss/

I've saved the final result in a public repository to use as a starting point for future projects. You can find it here:

- https://github.com/l4sh/hugo-tailwind-starter

## Prerequisites

- [Hugo](https://gohugo.io)
- [Node.js](https://nodejs.org/en/) with npm or pnpm. In this case I'll be using pnpm

## Create a new Hugo site

```bash
hugo new site my-hugo-site
cd my-hugo-site
```

## Convert `hugo.toml` to `hugo.yaml`:

Since the file was short I just did it manually, but later found out that it can be done with the following command:


```bash
hugo config --format yaml > hugo.yaml
```

## Install TailwindCSS

```bash
pnpm init
pnpm add -D tailwindcss @tailwindcss/cli
```

## Update `hugo.yaml`

```yaml
build:
  buildStats:
    enable: true
  cachebusters:
  - source: assets/notwatching/hugo_stats\.json
    target: css
  - source: (postcss|tailwind)\.config\.js
    target: css
module:
  mounts:
  - source: assets
    target: assets
  - disableWatch: true
    source: hugo_stats.json
    target: assets/notwatching/hugo_stats.json
```

## Create or update main CSS file

Add the following to `assets/css/main.css`:

```css
@import "tailwindcss";
@source "hugo_stats.json";
```

## Add CSS partial

Create a new file `layouts/partials/css.html` and add the following:

```html
{% raw %}
{{ with (templates.Defer (dict "key" "global")) }}
  {{ with resources.Get "css/main.css" }}
    {{ $opts := dict
      "minify" hugo.IsProduction
      "inlineImports" true
    }}
    {{ with . | css.TailwindCSS $opts }}
      {{ if hugo.IsProduction }}
        {{ with . | fingerprint }}
          <link rel="stylesheet" href="{{ .RelPermalink }}" integrity="{{ .Data.Integrity }}" crossorigin="anonymous">
        {{ end }}
      {{ else }}
        <link rel="stylesheet" href="{{ .RelPermalink }}">
      {{ end }}
    {{ end }}
  {{ end }}
{{ end }}
{% endraw %}
```

## Add a tailwind.config.js file

If using Tailwind CSS IntelliSense in VSCode create a new file `tailwind.config.js` and add the following:

```javascript
/*
  This file is to avoid issues with tailwind intellisense in VSCode
*/
```

## Add partials and baseof

Up to this point it's been pretty much the same as the official guide on adding Tailwind.
Next we need to add some partials that will allow to manage parts of the template more easily.

### Add meta

Create a new file `layouts/partials/meta.html` and add the following:

```html
{% raw %}
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="description" content="{{ .Site.Params.description }}">
<title>{{ .Site.Title }}{{ with .Title }} | {{ . }}{{ end }}</title>

<meta property="og:title" content="{{ .Site.Title }}{{ with .Title }} | {{ . }}{{ end }}">
<meta property="og:description" content="{{ .Site.Params.description }}">
<meta property="og:url" content="{{ .Permalink }}">
<meta property="og:type" content="website">
<meta property="og:image" content="{{ .Site.Params.socialImage }}">

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{{ .Site.Title }}{{ with .Title }} | {{ . }}{{ end }}">
<meta name="twitter:description" content="{{ .Site.Params.description }}">
<meta name="twitter:image" content="{{ .Site.Params.socialImage }}">
<meta name="twitter:site" content="{{ .Site.Params.twitterHandle }}">
{% endraw %}
```

### Add js partial
Create a new file `layouts/partials/js.html` and add the following:

```html
{% raw %}
{{ $jsFiles := resources.Match "js/*.js" }}
{{ $bundle := $jsFiles | resources.Concat "js/bundle.js" | resources.Minify }}
<script src="{{ $bundle.RelPermalink }}" defer></script>
{% endraw %}
```

This would allow us to add JS files in the `assets/js` folder and they will be automatically concatenated and minified.

### Add nav partial

Create a new file `layouts/partials/nav.html` and add the following:

```html
<nav class="bg-gray-800 text-white p-4">
  <ul class="flex space-x-4">
    <li><a href="/" class="hover:underline">Home</a></li>
    <li><a href="/about" class="hover:underline">About</a></li>
    <li><a href="/contact" class="hover:underline">Contact</a></li>
  </ul>
</nav>
```

### Add header partial


Create a new file `layouts/partials/header.html` and add the following:

```html
{% raw %}
<header class="bg-gray-800 text-white p-4">
  <h1 class="text-2xl font-bold">My Website</h1>
  {{ partial "nav.html" . }}
</header>
{% endraw %}
```


### Add footer
Create a new file `layouts/partials/footer.html` and add the following:

```html
<footer class="bg-gray-800 text-white p-4 text-center">
  <p class="text-sm">&copy; Your Website. All rights reserved.</p>
</footer>
```

### Add baseof
Finaly create a new file `layouts/_default/baseof.html` and add the following:

```html
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
  {{ partial "meta.html" . }}
  {{ partial "css.html" . }}
</head>
<body class="flex flex-col min-h-screen">
  {{ partial "header.html" . }}
  <main class="flex-grow">
    {{ block "main" . }}{{ end }}
  </main>
  {{ partial "footer.html" . }}
  {{ partial "js.html" . }}
</body>
</html>
{% endraw %}
```

This ties everything together and provides a base layout for all pages.

## Add a 404 page
Create a new file `layouts/404.html` and add the following:

```html
{% raw %}
{{ define "main" }}
  <div class="flex flex-col items-center justify-center min-h-screen bg-gray-100 text-center">
    <h1 class="text-4xl font-bold text-gray-800 mb-4">Page Not Found</h1>
    <p class="text-lg text-gray-600 mb-6">Sorry, the page you are looking for does not exist.</p>
    <a href="/" class="text-blue-500 hover:underline text-lg">Go back to the homepage</a>
  </div>
{{ end }}
{% endraw %}
```

## Add an index page

Now we create a new index page that we'll use as a test page to make sure tailwind
is being loaded and the layout is working properly.

Create a new layout file `layouts/index.html` and add the following:

```html
{% raw %}
{{ define "main" }}
  <div class="p-8">
    <h1 class="text-4xl font-bold mb-4">Tailwind CSS test</h1>
    <p class="text-lg mb-4">This page demonstrates various Tailwind CSS styles.</p>

    <h2 class="text-2xl font-semibold mt-6 mb-2">Typography</h2>
    <p class="mb-2">This is a <span class="text-blue-500">blue text</span> example.</p>
    <p class="italic mb-2">This text is italicized.</p>
    <p class="font-bold mb-2">This text is bold.</p>

    <h2 class="text-2xl font-semibold mt-6 mb-2">Buttons</h2>
    <button class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">Primary Button</button>
    <button class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600 ml-2">Secondary Button</button>

    <h2 class="text-2xl font-semibold mt-6 mb-2">Forms</h2>
    <form class="space-y-4">
      <input type="text" placeholder="Text input" class="border border-gray-300 p-2 rounded w-full">
      <select class="border border-gray-300 p-2 rounded w-full">
        <option>Option 1</option>
        <option>Option 2</option>
      </select>
      <button type="submit" class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600">Submit</button>
    </form>

    <h2 class="text-2xl font-semibold mt-6 mb-2">Cards</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <div class="border border-gray-300 p-4 rounded shadow">
        <h3 class="font-bold text-lg mb-2">Card Title</h3>
        <p class="text-gray-700">This is a simple card example.</p>
      </div>
      <div class="border border-gray-300 p-4 rounded shadow">
        <h3 class="font-bold text-lg mb-2">Card Title</h3>
        <p class="text-gray-700">This is another card example.</p>
      </div>
    </div>
  </div>
{{ end }}
{% endraw %}
```

The end result should look something like this:

![Hugo with TailwindCSS](assets/img/posts/hugo-tailwind.png)