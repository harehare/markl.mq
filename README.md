<div align="center">
  <img src="assets/logo.svg" style="width: 128px; height: 128px;"/>

<h1>markl.mq</h1>

**A Markdown template engine written in [mq](https://github.com/harehare/mq).**

</div>

markl.mq renders text with `{{ expression }}` tags evaluated by [mqmq](https://github.com/harehare/mqmq), an mq interpreter written in mq.

## Features

- Expressions, conditionals, loops, and `{% set %}` bindings: `{{ title | h1 }}`, `{% if %}`, `{% for %}`.
- Partials, components, and layouts with `{% include %}`, `{% component %}`, and `{% extends %}`.
- Themes: render one article for GitHub, Zenn, or dev.to by switching the `theme` option.
- Articles with YAML front matter, a site-wide `pages` index, and whole-directory builds.
- Mermaid diagrams and HTML output.
- Compile once, render many times; errors report source, line, and column.

## Quick start

Clone with submodules (markl imports [mqmq](https://github.com/harehare/mqmq) and [mermaid.mq](https://github.com/harehare/mermaid.mq)):

```sh
git clone --recurse-submodules <repository-url>
```

From the `markl.mq` directory, include `marl.mq` and pass a template and data to `marl_render`:

```sh
mq -I null -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_render("# {{ title }}\n{% for item in items %}- {{ item }}\n{% endfor %}", {title: "Notes", items: ["one", "two"]})'
```

```markdown
# Notes

- one
- two
```

## Documentation

| Page | Contents |
| --- | --- |
| [Getting started](docs/getting-started.md) | Installation, template files, pipelines, `.mqc` compile, tests |
| [Template syntax](docs/syntax.md) | Tags, variables, whitespace control, partials |
| [Filters and helpers](docs/filters.md) | Markdown formatting, `table`, `bullets`, `toc`, `date`, and more |
| [Layouts, components, and themes](docs/layouts-and-themes.md) | `include ... with`, `component`, `extends`, `theme` |
| [Articles and site builds](docs/articles.md) | Front matter, `marl_build`, Mermaid, HTML output |
| [API](docs/api.md) | Functions, options, file helpers, `marl_build_site` |

See [`examples/`](examples) for an article rendered with three themes (`default`, `zenn`, `devto`), and [`blog/`](blog) for a complete bilingual blog about mq built with markl.mq.

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.10.0 or later and [mqmq](https://github.com/harehare/mqmq) (included as the `vendor/mqmq` submodule).

## License

MIT
