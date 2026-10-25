# Template syntax

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

Expression results use `to_string`; `None` renders as an empty string. Markl does not HTML-escape inserted values.

Expressions follow mqmq's supported query language. See [mqmq's README](https://github.com/harehare/mqmq#known-limitations) for its language coverage and limitations.

## Data and variables

The data dictionary's keys are available as bare names. `.` starts as the data value and becomes the current item inside a `for` block. The loop variable is also available by name. `iteration` provides `index` (starting at 1), `index0` (starting at 0), `first`, `last`, and `length`:

```text
{% for item in items %}{{ iteration[:index] }}. {{ item }}
{% endfor %}
```

The `env` option adds bindings and overrides matching data keys. `{% set %}` bindings remain available later in the same scope, including after an `if` branch. Bindings created inside a `for` block stay inside that iteration.

The `for` block's `else` branch runs only for an empty array. It uses the outer data value and bindings; values set there remain available after the loop.

Pipes work inside call arguments, so `{{ join(tags | map(fn(t): upper(t) end), ", ") }}` is valid. Pipes still cannot appear inside array items or dict values (as in mq); wrap them in a call or parentheses.

## Undefined variables

By default a name that is not in the data or environment is an error that points at the tag (`Undefined variable 'name'`). Use `data[:key] | default(...)` for optional values, or pass the `lenient: true` option to treat missing names as `None` instead, so they render as an empty string and are falsy in `if`. Calling an unknown function is always an error.

## Whitespace control

Add `-` inside a tag's delimiter to strip the whitespace (including newlines) on that side: `{{-`, `{%-`, and `{#-` strip before the tag, and `-}}`, `-%}`, and `-#}` strip after it. This keeps block tags from leaving blank lines in the output:

```text
{% for item in items -%}
- {{ item }}
{% endfor -%}
```

## Raw blocks

Use `raw` to show template syntax in Markdown examples:

```text
{% raw %}
Use {{ name }} in your own template.
{% endraw %}
```

## Partials

`{% include "name" %}` renders another template with the current data and bindings. Partials come from the `partials` option, a dict of name to template string (markl does not read files). Includes can nest up to 16 levels, and `set` bindings inside a partial do not leak out:

```sh
mq -I null -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_render("{% include \"head\" %}body", {title: "T"}, {source_name: "page.md", partials: {head: "# {{ title }}\n"}})'
```

For layouts, components, and themes built on partials, see [Layouts, components, and themes](layouts-and-themes.md).
