# Go Frontend App Router

This projects aims to create a simple frontend app router like Next.js.

## How it works

All routes are defined in the `app` directory. The main router instance will handle the request and render the matching `page.html` file.

All routing depends on the directory structure. No more manual routing configuration is needed.

Additional middleware can be added to the router instance.

## Features

### `page.html`

The content will be rendered in the matching url. Unless using with a `page.go` file, it will be rendered as a _static page_. Examples:

| URL                         | Rendered file                            |
| --------------------------- | ---------------------------------------- |
| `/contacts`                 | `app/contacts/page.html`                 |
| `/panel/dashboard`          | `app/panel/dashboard/page.html`          |
| `/panel/dashboard/settings` | `app/panel/dashboard/settings/page.html` |

### `layout.html`

Optional. But recommended. At least a root `layout.html` is will be fine to include `<html>` and `<body>` tags. Can be used to wrap the `page.html` file. An outer `layout.html` will wrap the inner `layout.html` and so on until the root `layout.html`. If there is no `layout.html` file at all, the `page.html` file will be rendered as is.

If you have a file structure like this:

```plaintext
app/
  - layout.html
  - page.html
  panel/
    - layout.html
    - page.html
    dashboard/
      - page.html
```

The requested url `/panel/dashboard` will render the `app/panel/dashboard/page.html` by wrapping it in the `app/panel/layout.html` and then wrapping it in the `app/layout.html`.

### Routes wrapped in `[]`

They can be used to define route parameters. Your route can have multiple parameters. Just be sure all parameters are unique on a route.

If you have a file structure like this:

```plaintext
app/
  - layout.html
  - page.html
  blogs/
    - layout.html
    - page.html
    [category]/
      - page.html
      [slug]/
        - page.html
```

The requested url `/blogs/programming/hello-world` will render the `app/blogs/[category]/[slug]/page.html` by wrapping it in the `app/blogs/[category]/layout.html`. The layout wrapping will also be applied.

### `page.go`

Optional. Only required for _dynamic pages_. Can be used to fetch data and pass it to the `page.html` file. Decides how the page will be rendered.

You may want to use a `page.go` file to fetch data and pass it to the `page.html` file. Especially useful with router parameters. But you can also use it on static urls with dynamic data.

Example 1 (static url with dynamic data):

```plaintext
app/
  - layout.html
  - page.html
  blogs/
    - layout.html
    - page.html
    - page.go
```

Here, the `/blogs` url will render the `app/blogs/page.html` file. It is a static route but it may contain dynamic data. You may have diferent blogs in different times. You can use a `page.go` file to fetch the data and pass it to the `page.html` file.

Example 2 (dynamic url with dynamic data):

```plaintext
app/
  - layout.html
  - page.html
  blogs/
    - layout.html
    - page.html
    [category]/
      - page.html
      - page.go
      [slug]/
        - page.html
        - page.go
```

Here, the `/blogs/programming/hello-world-in-go` url will render the `app/blogs/[category]/[slug]/page.html` file. It is a dynamic route. You would probably use a `page.go` file to fetch the data related to the `programming` category and the `hello-world-in-go` slug. and pass it to the `page.html` file.

We will be explained later how to use the `page.go` file in the next sections.

### `static.json`

Optional. A structured json file that can be used to generate static data to be passed to the `page.html` or `layout.html` file. No extra configuration is needed. All pages will inherit the static data from the parent directories recursively. Inner data will override outer data.

Example:

```json
{
  "breadcrumbs": [
    { "url": "/", "text": "Home" },
    { "url": "/settings", "text": "Settings" },
    { "text": "Profile" }
  ]
}
```

### `layout.go`

Optional. Only required for _dynamic layouts_. Can be used to fetch data and pass it to the `layout.html` file. Decides how the layout will be rendered.

You may want to use a `layout.go` file to fetch data from an API or just from `static.json` file.

We will be explained later how to use the `layout.go` file in the next sections.

### `not-found.html`

Optional. If a `page.go` file decide to throw a 404, the `not-found.html` file will be rendered. If there is no `not-found.html` file, the router will try to find one in its parent directories recursively. If not found, the router will render a standard 404 page. For example, if you have a file structure like this:

```plaintext
app/
  - layout.html
  - page.html
  - not-found.html
  blogs/
    - layout.html
    - page.html
    - page.go
    [category]/
      - page.html
      - page.go
      [slug]/
        - not-found.html
        - page.html
        - page.go
```

User requests `/blogs/programming/hello-world-in-go`. The `app/blogs/[category]/[slug]/page.html` file will be rendered. The `/blogs/[category]/[slug]/page.go` tried to fetch the data for the `programming` category and the `hello-world-in-go` slug. But it failed. The router will try to find a `not-found.html` file in the `app/blogs/[category]/[slug]` directory. Since it exists, the `app/blogs/[category]/[slug]/not-found.html` file will be rendered.

User requests `/blogs/programming`. The `app/blogs/[category]/page.html` file will be rendered. The `/blogs/[category]/page.go` tried to fetch the data for the `programming` category. But it failed. The router will try to find a `not-found.html` file in the `app/blogs/[category]` directory. Since
there's no `not-found.html` file, the router will try to find one in the parent directories recursively. the first `not-found.html` file found will be rendered.

Only the found `not-found.html`'s layouts will be applied. Not the requested page's layouts.

### `error.html`

Just like `not-found.html`, but for other errors. You can use `_error.status` and `_error.message` to get the error code and the error message. A recursive search will be performed to find the `error.html` file.

### `metadata.json`

Optional. A structured json file that can be used to generate meta tags to be injected in the `<head>` tag. No extra configuration is needed. All pages will inherit the metadata from the parent directories recursively. Inner data will override outer data.

Example:

```json
{
  "title": "My Blog",
  "description": "This is my blog",
  "keywords": ["blog", "programming", "go"],
  "robots": "noindex, nofollow"
}
```

The `title` will be used as the `<title>` tag. The `description` will be used as the `<meta name="description" content="...">` tag. The `keywords` will be used as the `<meta name="keywords" content="...">` tag. The `robots` will be used as the `<meta name="robots" content="...">` tag.

We will be explained later all the features in the next sections.

### `metadata.go`

Optional. Only required for _dynamic metadata_. You can use it to fetch data according to dynamic routes or frequently changed static routes. You can pass dynamic data to override the existing `metadata.json` file or create it from scratch.

We will be explained later how to use the `metadata.go` file in the next sections.

### Routes wrapped in `()`

They can be used to group routes and prevent them from being use the sibling layouts, metadata, etc.

If you have a file structure like this:

```plaintext
app/
  - layout.html
  auth/
    - layout.html
    - page.html
    login/
      - page.html
  (panel)/
    - layout.html
    - page.html
    settings/
      - page.html
```

The requested url `/auth/login` will render the `app/auth/login/page.html` file. The `(panel)` route is wrapped in `()`. So, the `/auth/*` pages will not use the `app/(panel)/layout.html` layout. Only the matching urls in `(panel)` will use it.

What if you have a file structure like this:

```plaintext
app/
  - layout.html
  (auth)/
    - layout.html
    - page.html
  (panel)/
    - layout.html
    - page.html
```

This is not a valid file structure. If the use requests for the `/` url, the router will try to find a `app/page.html` file. But it will not find it. So, the router will try to find one in grouped routes. There's two grouped routes that have a `page.html` file: `(auth)` and `(panel)`.

In this case, the router will use the first one (alphabetically). So, you would never able to render the `app/(panel)/page.html` file.

### `middleware.go`

Optional. Recursively applies to inner routes. Applies specific rules to routes. For example, you can use it to check if the user is authenticated or not.

Example:

```plaintext
app/
  - middleware.go
  auth/
    - layout.html
    login/
      - page.html
  panel/
    - layout.html
    - page.html
```

This file can get the request url and the request method. By adding a `middleware.go` file to the `app` directory for authentication purposes, you can check if the user is authenticated or not. You can mark pages start with `panel` as protected. If the user is not authenticated, the router can redirect to the `/auth/login` page.

### `increment.go`

Optional. Only required for _dynamic pages_. You can create an interval to fetch data from an API or filesystem. Pages with `increment.go` will generate a static html file. A cron will be created to fetch the data and generate a new and updated static html file at runtime.

Example:

```plaintext
app/
  - increment.go
  - page.html
```

This project has a simple main page. it will fetch the data from an API and generate a static html file. It may be a "My youtube videos" page. Let's say the data is fetched every 120 minutes. You can create an `increment.go` file to fetch the data every 120 minutes.

## Static Builds

App router can detect the static pages at build time. It will be used to generate the static files. Pages without dynamic data can be generated as a static html file to serve without any additional processing.

## Special directories in order to `app` directory

### `public`

Optional. A directory that can be used to serve static files. It will be served as is. No additional processing is applied. A simple file server.
