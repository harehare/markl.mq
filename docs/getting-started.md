# Getting started

## Installation

Markl imports [mqmq](https://github.com/harehare/mqmq) and [mermaid.mq](https://github.com/harehare/mermaid.mq) as external modules, vendored as git submodules in `vendor/mqmq` and `vendor/mermaid.mq`. Clone with submodules:

```sh
git clone --recurse-submodules <repository-url>
# or, in an existing clone
git submodule update --init
```

Run the commands below from the `markl.mq` directory. The `-L vendor/mqmq -L vendor/mermaid.mq` options let `import "mqmq"` and `import "mermaid"` resolve.

## Usage

Include `marl.mq`, then pass a template string and a data value to `marl_render`:

```sh
mq -I null -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | marl_render("# {{ title }}\n{% for item in items %}- {{ item }}\n{% endfor %}", {title: "Notes", items: ["one", "two"]})'
```

```markdown
# Notes

- one
- two
```

## Render a template file

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
mq -L . -L vendor/mqmq -L vendor/mermaid.mq --rawfile template page.md \
  'include "marl" | marl_render(template, .)' data.json
```

## Use a pipeline

`marl(template)` renders against the current pipeline value (`self`):

```sh
mq -I null -L . -L vendor/mqmq -L vendor/mermaid.mq \
  'include "marl" | {name: "Ada"} | marl("Hello, {{ name }}!")'
# => Hello, Ada!
```

## Compile to a single `.mqc` program

Bundle `marl.mq` and mqmq into one `markl.mqc` program that runs without `-L`:

```sh
mq compile -I json -L . -L vendor/mqmq -L vendor/mermaid.mq -o markl.mqc \
  'include "marl" | marl_render(template, .)'

mq --rawfile template page.md markl.mqc data.json
```

The `.mqc` format is experimental, so recompile after upgrading mq.

## Tests

Tests are run with `mq-test`. From the `markl.mq` directory, run:

```sh
mq-test -L vendor/mqmq -L vendor/mermaid.mq marl_tests.mq
```

`-L` requires a `mq-test` release that supports `--directory`.

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.10.0 or later and [mqmq](https://github.com/harehare/mqmq) (included as the `vendor/mqmq` submodule). Template expressions use mqmq's supported language features.
