# Articles and site builds

## Pages

`marl_render_page(text, data, options)` treats `text` as a Markdown page:

1. The leading `---` YAML block is parsed (see below) and available as `page`, for example `{{ page[:title] }}`.
2. The body is rendered as a template with `data` and `page`.
3. The result is passed to the layout as `content`. The layout is `page[:layout]`, the `layout` option, or a partial named `layout` when one exists (`layout: false` skips it). Layouts can themselves `extend` a base layout.

Fenced code blocks (``` or `~~~`) and inline code spans are not interpreted as templates in pages, so articles can show `{{ ... }}`, `${{ ... }}` and `{% ... %}` syntax verbatim. Set `literal_code: false` to evaluate tags inside code, or use `{% raw %}` in plain `marl_render` calls (where `literal_code` defaults to `false`). A code span ends at the next run of the same number of backticks, and stops at a blank line; an unclosed fence runs to the end of the text.

The `theme` option wins over the page's own `theme` front matter key, so one article can be built once per platform. Front matter supports the YAML subset commonly used by blog platforms: `key: scalar` (strings, numbers, booleans, `~`/`null`), quoted strings, inline arrays, block lists of scalars, nested mappings, and `|` / `>` block scalars. Everything else is kept as a string. `marl_parse_page(text)` returns `{meta, body}`.

## Building many pages

`marl_render_pages` and `marl_build` render many pages. Every page sees `pages`, the front matter of all pages (newest `date` first) with `name` and `slug` added, so layouts can show related articles or an index. `page` also carries `name` and `slug` there.

```sh
# content/*.md hold articles, themes/<theme>/*.md hold layouts and components
mq -I null -R -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_build("examples/content", "examples/themes", {}, {theme: ["zenn", "default"]})'
```

One failing page normally stops the whole build with an error naming the file. To build everything that can be built and see all the errors at once, use the `try` variants:

```sh
mq -I null -R -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | let r = marl_try_build("content", "themes", {}, {theme: "zenn"}) | r[:errors]'
```

Writing the result to disk goes through `marl_write_all`, which needs existing directories (see [API](api.md#read-and-write-files)).

## Mermaid diagrams

With the `mermaid` option, `marl_render_page` replaces every ```` ```mermaid ```` block of the finished page using [mermaid.mq](https://github.com/harehare/mermaid.mq): `mermaid: true` (or `"svg"`) inlines an SVG, `mermaid: "text"` draws a text block. Blocks that cannot be drawn stay as they are. It is off by default, since Zenn and dev.to render Mermaid themselves.

```sh
mq -I null -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_render_page("```mermaid\ngraph LR\n  A --> B\n```", {}, {mermaid: true})'
```

## HTML output

With `html: true`, `marl_render_page` converts the finished Markdown to HTML (`to_html`) and wraps it in the theme's `html` partial, a full document with the theme's CSS. The partial receives `content` (the HTML body), `page` and the page data. `examples/themes/{default,zenn,devto}/html.md` follow the same `theme` lookup as layouts, including fallbacks such as `theme: ["zenn", "default"]`, and support light and dark mode. Without an `html` partial the bare HTML is returned.

`mermaid` blocks are drawn after the conversion, so the SVG is inlined as is. `mermaid_theme` (`"light"`, `"dark"` or `"auto"`, default `"light"`) sets the diagram theme; use `"auto"` to follow the page's `prefers-color-scheme`.

```sh
mq -I null -R -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_build("examples/content", "examples/themes", {}, {theme: "default", html: true, mermaid: true, mermaid_theme: "auto"})'
```

Templates can read `html` (true with `html: true`), so a component can emit HTML instead of platform syntax: the `zenn` callout renders a `<div class="message">` for HTML output and `:::message` otherwise.
