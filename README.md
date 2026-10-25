<div align="center">
  <img src="assets/logo.svg" style="width: 128px; height: 128px;"/>

<h1>markl.mq</h1>

**A Markdown template engine written in [mq](https://github.com/harehare/mq).**

</div>

markl.mq renders text with `{{ expression }}` tags evaluated by [mqmq](https://github.com/harehare/mqmq), an mq interpreter written in mq.

## Features

- Insert expression results with `{{ expression }}`.
- Use `{% if %}`, `{% elif %}`, and `{% else %}` for conditional content.
- Repeat content with `{% for name in array %}`, render an `{% else %}` branch for an empty array, and access loop positions through `iteration`.
- Bind values for later tags with `{% set name = expression %}`.
- Reuse partial templates with `{% include "name" %}`, pass them values with `with {...}`, or wrap Markdown in a `{% component %}`.
- Build layouts with `{% extends %}` and `{% block %}`, and switch between themes with the `theme` option.
- Render articles with YAML front matter, a layout, and a site-wide `pages` index; build a whole directory in one call.
- Article helpers: `toc`, `slugify`, `anchor`, `date`, `reading_time`, `details`, `image`, and more.
- Ignore `{# comments #}` or preserve literal template syntax inside `{% raw %}` blocks.
- Transform values with filters such as `{{ title | h1 }}`, `{{ plot | blockquote }}`, and `{{ tags | wikilink }}`.
- Generate YAML front matter with `frontmatter()`.
- Control whitespace with `{%- -%}`, `{{- -}}`, and `{#- -#}`.
- Compile a template once for repeated rendering.
- Report template source, line, and column for compile and render errors.

## Installation

Markl imports [mqmq](https://github.com/harehare/mqmq) as an external module, vendored as a git submodule in `vendor/mqmq`. Clone with submodules:

```sh
git clone --recurse-submodules <repository-url>
# or, in an existing clone
git submodule update --init
```

Run the commands below from the `markl.mq` directory. The `-L vendor/mqmq` option lets `import "mqmq"` resolve mqmq and its own modules.

## Usage

Include `marl.mq`, then pass a template string and a data value to `marl_render`:

```sh
mq -I null -L . -L vendor/mqmq \
  'include "marl" | marl_render("# {{ title }}\n{% for item in items %}- {{ item }}\n{% endfor %}", {title: "Notes", items: ["one", "two"]})'
```

```markdown
# Notes

- one
- two
```

### Render a template file

Given `page.md`:

```text
# {{ title }}
{% for item in items %}- {{ item }}
{% endfor %}
```

and `data.json`:

```json
{ "title": "Notes", "items": ["one", "two"] }
```

load the template as a raw string and pass each JSON input value as data:

```sh
mq -L . -L vendor/mqmq --rawfile template page.md \
  'include "marl" | marl_render(template, .)' data.json
```

### Use a pipeline

`marl(template)` renders against the current pipeline value (`self`):

```sh
mq -I null -L . -L vendor/mqmq \
  'include "marl" | {name: "Ada"} | marl("Hello, {{ name }}!")'
# => Hello, Ada!
```

### Compile to a single `.mqc` program

Bundle `marl.mq` and mqmq into one `markl.mqc` program that runs without `-L`:

```sh
mq compile -I json -L . -L vendor/mqmq -o markl.mqc \
  'include "marl" | marl_render(template, .)'

mq --rawfile template page.md markl.mqc data.json
```

The `.mqc` format is experimental, so recompile after upgrading mq.

## Template syntax

| Syntax                                                                   | Effect                                                                                       |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| `{{ expression }}`                                                       | Evaluate an mqmq expression and insert its result                                            |
| `{# comment #}`                                                          | Ignore the comment                                                                           |
| `{% if expression %}...{% elif expression %}...{% else %}...{% endif %}` | Render the first matching branch                                                             |
| `{% for name in expression %}...{% else %}...{% endfor %}`               | Render once for each item in an array, or render the optional `else` branch when it is empty |
| `{% set name = expression %}`                                            | Bind a value for later tags                                                                  |
| `{% include "name" %}`                                                   | Render the partial `name` (from the `partials` argument) with the current data and bindings  |
| `{% include "name" with {key: value} %}`                                 | Same, with extra bindings that exist only inside the partial                                 |
| `{% component "name" with {...} %}...{% endcomponent %}`                 | Render the partial with the rendered block contents available as `body`                      |
| `{% capture name %}...{% endcapture %}`                                  | Bind the rendered contents to `name`                                                         |
| `{% extends "name" %}` / `{% block name %}...{% endblock %}`             | Render the parent template with this template's blocks replacing the parent's               |
| `{% raw %}...{% endraw %}`                                               | Insert the contents literally, without interpreting template tags                            |

### Whitespace control

Add `-` inside a tag's delimiter to strip the whitespace (including newlines) on that side: `{{-`, `{%-`, and `{#-` strip before the tag, and `-}}`, `-%}`, and `-#}` strip after it. This keeps block tags from leaving blank lines in the output:

```text
{% for item in items -%}
- {{ item }}
{% endfor -%}
```

### Filters

Expressions are mq, so the pipe operator works as a filter. Markdown formatting functions from mqmq are available:

| Filter | Result |
|---|---|
| `{{ title \| h1 }}` ... `h6` | `# Title` ... |
| `{{ plot \| blockquote }}` | `> plot` (per line) |
| `{{ name \| bold }}`, `italic`, `strike`, `code_inline` | `**name**`, `*name*`, `~~name~~`, `` `name` `` |
| `{{ code("rust") }}` | fenced code block around the current value |
| `{{ wikilink(tags) }}` | `[[tag]]` for a value or each item of an array |
| `{{ value \| yaml_property("key") }}` | one YAML property, `key: value` |

markl adds these helpers. The piped value is always the last argument:

| Helper | Result |
|---|---|
| `{{ x \| default("n/a") }}` | the fallback when `x` is `None` or `""` |
| `{{ text \| link("https://example.com") }}` | `[text](https://example.com)` |
| `{{ meta \| frontmatter() }}` | a `---` YAML front matter block from a dict; `None` values are skipped |
| `{{ text \| indent(2) }}` | indents every line except the first, so multi-line values stay inside list items |
| `{{ items \| bullets() }}` / `{{ items \| numbered() }}` | an array as a `- ` or `1. ` list; multi-line items are indented |
| `{{ rows \| table(["name", "age"]) }}` | a Markdown table; each row is an array, or a dict keyed by header. `\|` and newlines in cells are escaped |
| `{{ text \| escape_md() }}` | escapes Markdown syntax characters with a backslash |
| `{{ text \| truncate(80) }}` | cuts text to 80 characters and appends `…` |

Helpers for articles:

| Helper | Result |
|---|---|
| `{{ md \| toc(2, 3) }}` | a nested list linking the `##`–`###` headings of Markdown text (headings in code blocks are ignored) |
| `{{ title \| slugify }}` | a URL slug: lowercase, runs of other characters become `-`; non-ASCII letters are kept |
| `{{ heading \| anchor }}` | a GitHub-style heading anchor, as used by `toc` |
| `{{ "2026-10-25" \| date("%b %d, %Y") }}` | formats a `YYYY-MM-DD` or RFC 3339 date with a `strftime` format |
| `{{ text \| reading_time }}` / `word_count` | estimated minutes (200 words or 500 kana/kanji per minute, at least 1) / number of whitespace-separated words |
| `{{ text \| details("Summary") }}` | an HTML `<details>` block around the text |
| `{{ url \| image("alt text") }}` | `![alt text](url)` |
| `{{ name \| capitalize }}` | upper-cases the first character |

For reusable Markdown with arguments, use partials and components (see below).

```text
{{ {title: title, tags: tags, date: date} | frontmatter() }}# {{ title }}
```

Pipes work inside call arguments, so `{{ join(tags | map(fn(t): upper(t) end), ", ") }}` is valid. Pipes still cannot appear inside array items or dict values (as in mq); wrap them in a call or parentheses. A bare name missing from the data is an error (`Undefined variable 'name'`), so use `data[:key] | default(...)` for optional values, or render leniently (see below).

### Undefined variables

By default a name that is not in the data or environment is an error that points at the tag. Pass the `lenient: true` option to treat missing names as `None` instead, so they render as an empty string and are falsy in `if`. Calling an unknown function is always an error.

### Partials

`{% include "name" %}` renders another template with the current data and bindings. Partials come from the `partials` option, a dict of name to template string (markl does not read files). Includes can nest up to 16 levels, and `set` bindings inside a partial do not leak out:

```sh
mq -I null -L . -L vendor/mqmq \
  'include "marl" | marl_render("{% include \"head\" %}body", {title: "T"}, {source_name: "page.md", partials: {head: "# {{ title }}\n"}})'
```

Expression results use `to_string`; `None` renders as an empty string. Markl does not HTML-escape inserted values.

The data dictionary's keys are available as bare names. `.` starts as the data value and becomes the current item inside a `for` block. The loop variable is also available by name. `iteration` provides `index` (starting at 1), `index0` (starting at 0), `first`, `last`, and `length`:

```text
{% for item in items %}{{ iteration[:index] }}. {{ item }}
{% endfor %}
```

The `env` option adds bindings and overrides matching data keys. `{% set %}` bindings remain available later in the same scope, including after an `if` branch. Bindings created inside a `for` block stay inside that iteration.

The `for` block's `else` branch runs only for an empty array. It uses the outer data value and bindings; values set there remain available after the loop. Use `raw` to show template syntax in Markdown examples:

```text
{% raw %}
Use {{ name }} in your own template.
{% endraw %}
```

Expressions follow mqmq's supported query language. See [mqmq's README](https://github.com/harehare/mqmq#known-limitations) for its language coverage and limitations.

## Layouts, components, and themes

Partials resolve through the `partials` dict. Three tags build on them.

**Parameters.** `{% include "callout" with {type: "tip"} %}` binds `type` only while the partial renders. `{% component "callout" with {type: "tip"} %}Markdown{% endcomponent %}` also renders its contents and hands them to the partial as `body`, which is how a shared snippet wraps arbitrary Markdown. `{% capture name %}...{% endcapture %}` stores rendered text in a variable.

**Inheritance.** A template that starts with `{% extends "base" %}` renders `base` with its own `{% block name %}...{% endblock %}` sections replacing the parent's. Text outside blocks is ignored, apart from top-level `{% set %}` tags, which bind values for the parent. Chains of `extends` work, and the most derived template wins. A block always has default content in the parent.

**Themes.** The `theme` option is a name or an array of names in priority order. A template name such as `"callout"` is looked up as `theme/callout` for each theme, then as written, and each candidate is also tried with the `ext` suffix (`.md`). A theme can therefore replace just the partials it cares about and fall back to a base theme: `{theme: ["zenn", "default"]}`. `theme` is also bound inside templates (the first theme, or `None`). Because Markdown flavors differ (callouts, front matter, details), a theme is usually one layout plus a handful of components.

### Articles

`marl_render_page(text, data, options)` treats `text` as a Markdown page:

1. The leading `---` YAML block is parsed (see below) and available as `page`, for example `{{ page[:title] }}`.
2. The body is rendered as a template with `data` and `page`.
3. The result is passed to the layout as `content`. The layout is `page[:layout]`, the `layout` option, or a partial named `layout` when one exists (`layout: false` skips it). Layouts can themselves `extend` a base layout.

Fenced code blocks (``` or `~~~`) and inline code spans are not interpreted as templates in pages, so articles can show `{{ ... }}`, `${{ ... }}` and `{% ... %}` syntax verbatim. Set `literal_code: false` to evaluate tags inside code, or use `{% raw %}` in plain `marl_render` calls (where `literal_code` defaults to `false`). A code span ends at the next run of the same number of backticks, and stops at a blank line; an unclosed fence runs to the end of the text.

The `theme` option wins over the page's own `theme` front matter key, so one article can be built once per platform. Front matter supports the YAML subset commonly used by blog platforms: `key: scalar` (strings, numbers, booleans, `~`/`null`), quoted strings, inline arrays, block lists of scalars, nested mappings, and `|` / `>` block scalars. Everything else is kept as a string. `marl_parse_page(text)` returns `{meta, body}`.

`marl_render_pages` and `marl_build` render many pages. Every page sees `pages`, the front matter of all pages (newest `date` first) with `name` and `slug` added, so layouts can show related articles or an index. `page` also carries `name` and `slug` there.

One failing page normally stops the whole build with an error naming the file. To build everything that can be built and see all the errors at once, use the `try` variants:

```sh
mq -I null -R -L . -L vendor/mqmq \
  'include "marl" | let r = marl_try_build("content", "themes", {}, {theme: "zenn"}) | r[:errors]'
```

```sh
# content/*.md hold articles, themes/<theme>/*.md hold layouts and components
mq -I null -R -L . -L vendor/mqmq \
  'include "marl" | marl_build("examples/content", "examples/themes", {}, {theme: ["zenn", "default"]})'
```

[`examples/`](examples) contains an article and three themes (`default` for GitHub-style Markdown, `zenn`, `devto`): the same `mq-intro.md` renders with GitHub alerts, Zenn `:::message` blocks, or dev.to front matter depending on the `theme` option. Writing the result to disk goes through `marl_write_all`, which needs existing directories (see above).

## API

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

For repeated rendering, compile once:

```sh
mq -I null -L . -L vendor/mqmq \
  'include "marl" | let ast = marl_compile("Hello, {{ name }}!") | [marl_render_ast(ast, {name: "Ada"}), marl_render_ast(ast, {name: "Lin"})]'
# => ["Hello, Ada!", "Hello, Lin!"]
```

Pass the `source_name` option when compiling or rendering to identify the template in errors. Errors point to the opening tag, for example `page.md:3:1: Unknown function: missing`. Without a source name, errors use `<template>`.

### Render multiple templates

`marl_render_all` renders a dict of name to template string in one call. Every template is also available as a partial to the others (explicit `partials` win), and errors use the template's name, so a shared layout works like a static site generator:

```sh
mq -I null -L . -L vendor/mqmq \
  'include "marl" | marl_render_all({"index.md": "{% include \"head.md\" %}Home", "about.md": "{% include \"head.md\" %}About", "head.md": "# {{ site }}\n"}, {site: "Notes"})'
# => {"index.md": "# Notes\nHome", "about.md": "# Notes\nAbout", "head.md": "# Notes\n"}
```

`marl_render_each` renders one template for every item of an array, compiling it once.

#### Read and write files

These helpers use mq's `read_file`, `walk_files`, and `write_file`, which are disabled by default. Pass `--allow-read` (`-R`) to read and `--allow-write` (`-W`) to write; restrict them with `-R=PATH` if you like.

| Function | Description |
| --- | --- |
| `marl_render_files(paths, data, options = {})` | Read each path and render with `marl_render_all`, keyed by path |
| `marl_read_dir(dir, options = {})` | Read every file under `dir` ending with the `ext` option into a dict of relative path to text |
| `marl_render_dir(dir, data, options = {})` | Render every file under `dir` ending with the `ext` option (default `".md"`). Keys are paths relative to `dir`, so partials are included as `{% include "sub/head.md" %}` |
| `marl_write_all(outputs, dir)` | Write a dict of name to output below `dir` and return the written paths. Directories must already exist |

```sh
mq -I null -R -W -L . -L vendor/mqmq \
  'include "marl" | marl_render_dir("content", {site: "Notes"}) | marl_write_all(., "public")'
```

## Tests

Tests are run with `mq-test`. From the `markl.mq` directory, run:

```sh
mq-test -L vendor/mqmq marl_tests.mq
```

`-L` requires a `mq-test` release that supports `--directory`.

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.9.0 or later and [mqmq](https://github.com/harehare/mqmq) (included as the `vendor/mqmq` submodule). Template expressions use mqmq's supported language features.

## License

MIT
