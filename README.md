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
mq -I null -F text -L . -L vendor/mqmq \
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
mq -I json -F text -L . -L vendor/mqmq --rawfile template page.md \
  'include "marl" | marl_render(template, .)' data.json
```

### Use a pipeline

`marl(template)` renders against the current pipeline value (`self`):

```sh
mq -I null -F text -L . -L vendor/mqmq \
  'include "marl" | {name: "Ada"} | marl("Hello, {{ name }}!")'
# => Hello, Ada!
```

### Compile to a single `.mqc` program

`mq compile` bundles `marl.mq` and mqmq into one `markl.mqc` program. The compiled program runs without `-L`, so it works from any directory:

```sh
mq compile -I json -L . -L vendor/mqmq -o markl.mqc \
  'include "marl" | marl_render(template, .)'

mq -I json -F text --rawfile template page.md markl.mqc data.json
```

Pass the same `-I` to `mq compile` and when running the `.mqc` program. `-F` is a run-time option only. The `.mqc` format is experimental, so recompile after upgrading mq.

## Template syntax

| Syntax                                                                   | Effect                                                                                       |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| `{{ expression }}`                                                       | Evaluate an mqmq expression and insert its result                                            |
| `{# comment #}`                                                          | Ignore the comment                                                                           |
| `{% if expression %}...{% elif expression %}...{% else %}...{% endif %}` | Render the first matching branch                                                             |
| `{% for name in expression %}...{% else %}...{% endfor %}`               | Render once for each item in an array, or render the optional `else` branch when it is empty |
| `{% set name = expression %}`                                            | Bind a value for later tags                                                                  |
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

markl adds three helpers. The piped value is always the last argument:

| Helper | Result |
|---|---|
| `{{ x \| default("n/a") }}` | the fallback when `x` is `None` or `""` |
| `{{ text \| link("https://example.com") }}` | `[text](https://example.com)` |
| `{{ meta \| frontmatter() }}` | a `---` YAML front matter block from a dict; `None` values are skipped |

```text
{{ {title: title, tags: tags, date: date} | frontmatter() }}# {{ title }}
```

Pipes cannot appear inside call arguments (mqmq limitation); nest calls instead, for example `{{ join(wikilink(tags), ", ") }}`. A key missing from the data is an error, so use `data[:key] | default(...)` for optional values.

Expression results use `to_string`; `None` renders as an empty string. Markl does not HTML-escape inserted values.

The data dictionary's keys are available as bare names. `.` starts as the data value and becomes the current item inside a `for` block. The loop variable is also available by name. `iteration` provides `index` (starting at 1), `index0` (starting at 0), `first`, `last`, and `length`:

```text
{% for item in items %}{{ iteration[:index] }}. {{ item }}
{% endfor %}
```

An optional environment dictionary adds bindings and overrides matching data keys. `{% set %}` bindings remain available later in the same scope, including after an `if` branch. Bindings created inside a `for` block stay inside that iteration.

The `for` block's `else` branch runs only for an empty array. It uses the outer data value and bindings; values set there remain available after the loop. Use `raw` to show template syntax in Markdown examples:

```text
{% raw %}
Use {{ name }} in your own template.
{% endraw %}
```

Expressions follow mqmq's supported query language. See [mqmq's README](https://github.com/harehare/mqmq#known-limitations) for its language coverage and limitations.

## API

| Function                                                            | Description                                     |
| ------------------------------------------------------------------- | ----------------------------------------------- |
| `marl_render(template, data, env = {}, source_name = "<template>")` | Compile and render a template to a string       |
| `marl_compile(template, source_name = "<template>")`                | Parse a template into a reusable AST            |
| `marl_render_ast(ast, data, env = {})`                              | Render a compiled template with new data        |
| `marl(template)`                                                    | Render using the current pipeline value as data |

For repeated rendering, compile once:

```sh
mq -I null -F text -L . -L vendor/mqmq \
  'include "marl" | let ast = marl_compile("Hello, {{ name }}!") | [marl_render_ast(ast, {name: "Ada"}), marl_render_ast(ast, {name: "Lin"})]'
# => ["Hello, Ada!", "Hello, Lin!"]
```

Pass `source_name` when compiling or rendering to identify the template in errors. Errors point to the opening tag, for example `page.md:3:1: Unknown function: missing`. Without a source name, errors use `<template>`.

## Tests

From the `markl.mq` directory, run:

```sh
mq -e -I null -F text -L . -L vendor/mqmq -f run_tests.mq
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.9.0 or later and [mqmq](https://github.com/harehare/mqmq) (included as the `vendor/mqmq` submodule). Template expressions use mqmq's supported language features.

## License

MIT
