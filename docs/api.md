# API

## Functions

| Function                                         | Description                                     |
| ------------------------------------------------ | ----------------------------------------------- |
| `marl_render(template, data, options = {})`      | Compile and render a template to a string       |
| `marl_compile(template, options = {})`           | Parse a template into a reusable AST            |
| `marl_render_ast(ast, data, options = {})`       | Render a compiled template with new data        |
| `marl(template)`                                 | Render using the current pipeline value as data |
| `marl_render_all(templates, data, options = {})` | Render a dict of name to template; returns a dict of name to output |
| `marl_render_each(template, items, options = {})` | Render one template per item of an array; returns an array of outputs |
| `marl_parse_page(text)`                          | Split a document into `{meta, body}` from its YAML front matter |
| `marl_render_page(text, data, options = {})`     | Render a page body, then wrap it in its layout |
| `marl_render_pages(pages, data, options = {})`   | Render a dict of name to page text, with a shared `pages` index |
| `marl_build(content_dir, themes_dir, data, options = {})` | Read pages and themes from directories and render every page |
| `marl_try_render_pages(pages, data, options = {})` | Like `marl_render_pages`, but returns `{outputs, errors}` so one failing page does not stop the rest |
| `marl_try_build(content_dir, themes_dir, data, options = {})` | Like `marl_build`, with the same `{outputs, errors}` result |

## Options

Options is a dict; every key is optional:

| Option | Default | Description |
| --- | --- | --- |
| `env` | `{}` | Extra bindings; they override matching data keys (render only) |
| `source_name` | `"<template>"` | Template name used in error messages |
| `partials` | `{}` | Dict of name to template string for `{% include %}` (render only) |
| `lenient` | `false` | Treat undefined names as `None` instead of an error (render only) |
| `theme` | none | A theme name, or an array of names in priority order, used to resolve template names (render only) |
| `ext` | `".md"` | Extension tried when resolving template names, and the file extension read by `marl_read_dir`, `marl_render_dir` and `marl_build` |
| `layout` | `"layout"` | Layout partial for `marl_render_page` and `marl_render_pages`; `false` renders without one |
| `literal_code` | `false` (`true` for pages) | Leave fenced code blocks and inline code spans untouched, so examples such as `${{ secrets.TOKEN }}` need no `raw` block. Pages default to `true`; pass `false` to evaluate tags inside code |
| `mermaid` | none | `true` / `"svg"` inlines Mermaid diagrams as SVG, `"text"` as text blocks (pages only; see [Articles](articles.md#mermaid-diagrams)) |
| `mermaid_theme` | `"light"` | `"light"`, `"dark"` or `"auto"` (pages only) |
| `html` | `false` | Convert a page to HTML and wrap it in the theme's `html` partial (pages only; see [Articles](articles.md#html-output)) |
| `heading_ids` | `true` | With `html: true`, give every heading an `id` that matches the `toc` links; `false` leaves headings without one (pages only) |
| `skip_unpublished` | `false` | Leave out pages whose front matter has `draft: true` or `published: false`: they are not rendered, not written by `marl_build_site`, and not listed in `pages` (`marl_render_pages`, `marl_build`, `marl_build_site` only) |

## Compile once

For repeated rendering, compile once:

```sh
mq -I null -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | let ast = marl_compile("Hello, {{ name }}!") | [marl_render_ast(ast, {name: "Ada"}), marl_render_ast(ast, {name: "Lin"})]'
# => ["Hello, Ada!", "Hello, Lin!"]
```

## Errors

Pass the `source_name` option when compiling or rendering to identify the template in errors. Errors point to the opening tag, for example `page.md:3:1: Unknown function: missing`. Without a source name, errors use `<template>`.

## Render multiple templates

`marl_render_all` renders a dict of name to template string in one call. Every template is also available as a partial to the others (explicit `partials` win), and errors use the template's name, so a shared layout works like a static site generator:

```sh
mq -I null -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_render_all({"index.md": "{% include \"head.md\" %}Home", "about.md": "{% include \"head.md\" %}About", "head.md": "# {{ site }}\n"}, {site: "Notes"})'
# => {"index.md": "# Notes\nHome", "about.md": "# Notes\nAbout", "head.md": "# Notes\n"}
```

`marl_render_each` renders one template for every item of an array, compiling it once.

## Read and write files

These helpers use mq's `read_file`, `walk_files`, and `write_file`, which are disabled by default. Pass `--allow-read` (`-R`) to read and `--allow-write` (`-W`) to write; restrict them with `-R=PATH` and `-W=PATH` if you like (the directory given to `-W=PATH` must already exist). Creating directories uses mq's `mkdir`, which needs `--allow-write` too.

| Function | Description |
| --- | --- |
| `marl_render_files(paths, data, options = {})` | Read each path and render with `marl_render_all`, keyed by path |
| `marl_read_dir(dir, options = {})` | Read every file under `dir` ending with the `ext` option into a dict of relative path to text |
| `marl_render_dir(dir, data, options = {})` | Render every file under `dir` ending with the `ext` option (default `".md"`). Keys are paths relative to `dir`, so partials are included as `{% include "sub/head.md" %}` |
| `marl_write_all(outputs, dir)` | Write a dict of name to output below `dir`, creating the directories it needs, and return the written paths |
| `marl_build_site(content_dir, themes_dir, out_dir, data, options = {})` | Build every page like `marl_build` and write the result below `out_dir`. If any page fails, nothing is written and one error lists every failing page. With `html: true`, `.md` becomes `.html` in the file names. Returns the written paths |

```sh
mq -I null -R -W -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_render_dir("content", {site: "Notes"}) | marl_write_all(., "public")'
```

`marl_build_site` is the same for a whole site with themes and layouts:

```sh
mq -I null -R -W -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_build_site("content", "themes", "public", {site: "Notes"}, {theme: "default", html: true})'
```
