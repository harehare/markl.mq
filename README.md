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
- Reuse partial templates with `{% include "name" %}`.
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

## API

| Function                                         | Description                                     |
| ------------------------------------------------ | ----------------------------------------------- |
| `marl_render(template, data, options = {})`      | Compile and render a template to a string       |
| `marl_compile(template, options = {})`           | Parse a template into a reusable AST            |
| `marl_render_ast(ast, data, options = {})`       | Render a compiled template with new data        |
| `marl(template)`                                 | Render using the current pipeline value as data |
| `marl_render_all(templates, data, options = {})` | Render a dict of name to template; returns a dict of name to output |
| `marl_render_each(template, items, options = {})` | Render one template per item of an array; returns an array of outputs |

Options is a dict; every key is optional:

| Option | Default | Description |
| --- | --- | --- |
| `env` | `{}` | Extra bindings; they override matching data keys (render only) |
| `source_name` | `"<template>"` | Template name used in error messages |
| `partials` | `{}` | Dict of name to template string for `{% include %}` (render only) |
| `lenient` | `false` | Treat undefined names as `None` instead of an error (render only) |

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
