---
title: "mq: jq for Markdown"
date: 2026-10-25
emoji: 🔍
published: false
description: Query and transform Markdown from the command line.
tags: [markdown, cli, rust]
---

{% set install = "cargo install mq-run" %}
## What is mq?

mq is a command-line tool that treats Markdown as structured data, so you can select headings, code blocks and links with a jq-like query language.

{% component "callout" with {type: "tip"} %}
Everything on this page is rendered from one source with the `{{ theme | default("default") }}` theme.
{% endcomponent %}

## Install

{{ install | code("sh") }}

## Extract headings

```sh
mq '.h2' README.md
```

### Why not grep?

`grep` sees lines; mq sees headings, lists and code blocks.

{% component "callout" with {type: "warning"} %}
mq is pre-1.0, so the query language may still change.
{% endcomponent %}

## More articles
{% set others = filter(pages, fn(article): article[:slug] != page[:slug] end) -%}
{% for article in others %}
- {{ article[:title] | link(article[:slug] + ".md") }}{% else %}
No other articles yet.{% endfor %}
