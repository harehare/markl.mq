# Layouts, components, and themes

Partials resolve through the `partials` dict. Three tags build on them.

## Parameters

`{% include "callout" with {type: "tip"} %}` binds `type` only while the partial renders. `{% component "callout" with {type: "tip"} %}Markdown{% endcomponent %}` also renders its contents and hands them to the partial as `body`, which is how a shared snippet wraps arbitrary Markdown. `{% capture name %}...{% endcapture %}` stores rendered text in a variable.

## Inheritance

A template that starts with `{% extends "base" %}` renders `base` with its own `{% block name %}...{% endblock %}` sections replacing the parent's. Text outside blocks is ignored, apart from top-level `{% set %}` tags, which bind values for the parent. Chains of `extends` work, and the most derived template wins. A block always has default content in the parent.

## Themes

The `theme` option is a name or an array of names in priority order. A template name such as `"callout"` is looked up as `theme/callout` for each theme, then as written, and each candidate is also tried with the `ext` suffix (`.md`). A theme can therefore replace just the partials it cares about and fall back to a base theme: `{theme: ["zenn", "default"]}`. `theme` is also bound inside templates (the first theme, or `None`). Because Markdown flavors differ (callouts, front matter, details), a theme is usually one layout plus a handful of components.

[`examples/`](../examples) contains an article and three themes (`default` for GitHub-style Markdown, `zenn`, `devto`): the same `mq-intro.md` renders with GitHub alerts, Zenn `:::message` blocks, or dev.to front matter depending on the `theme` option.
