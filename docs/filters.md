# Filters and helpers

Expressions are mq, so the pipe operator works as a filter: `{{ title | h1 }}`, `{{ plot | blockquote }}`, `{{ tags | wikilink }}`.

## Markdown formatting (from mqmq)

| Filter | Result |
|---|---|
| `{{ title \| h1 }}` ... `h6` | `# Title` ... |
| `{{ plot \| blockquote }}` | `> plot` (per line) |
| `{{ name \| bold }}`, `italic`, `strike`, `code_inline` | `**name**`, `*name*`, `~~name~~`, `` `name` `` |
| `{{ code("rust") }}` | fenced code block around the current value |
| `{{ wikilink(tags) }}` | `[[tag]]` for a value or each item of an array |
| `{{ value \| yaml_property("key") }}` | one YAML property, `key: value` |

## markl helpers

The piped value is always the last argument.

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

Generate front matter from a dict:

```text
{{ {title: title, tags: tags, date: date} | frontmatter() }}# {{ title }}
```

## Article helpers

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

For reusable Markdown with arguments, use partials and components (see [Layouts, components, and themes](layouts-and-themes.md)).
