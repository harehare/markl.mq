{% raw %}<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{% endraw %}{{ page[:title] }}{% raw %}</title>
<style>
:root { --bg: #f5f5f5; --card: #ffffff; --fg: #171717; --muted: #525252; --border: #e5e5e5; --code-bg: #f5f5f5; --link: #3b49df; }
@media (prefers-color-scheme: dark) {
  :root { --bg: #0a0a0a; --card: #171717; --fg: #f5f5f5; --muted: #a3a3a3; --border: #333333; --code-bg: #262626; --link: #8d95ff; }
}
body { margin: 0; padding: 24px 12px; background: var(--bg); color: var(--fg);
  font: 18px/1.7 -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
article { max-width: 760px; margin: 0 auto; padding: 32px 48px; background: var(--card); border-radius: 8px; box-shadow: 0 0 0 1px var(--border); }
h1 { font-size: 2.2em; line-height: 1.2; }
a { color: var(--link); }
pre, code { background: var(--code-bg); border-radius: 6px; font-family: ui-monospace, Menlo, monospace; }
code { padding: .1em .3em; font-size: 85%; }
pre { padding: 16px; overflow: auto; font-size: 15px; }
pre code { padding: 0; background: none; }
blockquote { margin: 0; padding: 0 1em; color: var(--muted); border-left: 4px solid var(--border); }
img, svg { max-width: 100%; height: auto; }
@media (max-width: 600px) { article { padding: 20px; } }
</style>
</head>
<body>
<article>
{% endraw %}{{ content }}{% raw %}
</article>
</body>
</html>
{% endraw %}
