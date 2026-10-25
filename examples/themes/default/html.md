{% raw %}<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{% endraw %}{{ page[:title] }}{% raw %}</title>
<style>
:root { --bg: #ffffff; --fg: #1f2328; --muted: #59636e; --border: #d1d9e0; --code-bg: #f6f8fa; --link: #0969da; }
@media (prefers-color-scheme: dark) {
  :root { --bg: #0d1117; --fg: #e6edf3; --muted: #9198a1; --border: #3d444d; --code-bg: #151b23; --link: #4493f8; }
}
body { max-width: 860px; margin: 0 auto; padding: 32px 16px; background: var(--bg); color: var(--fg);
  font: 16px/1.6 -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", Helvetica, Arial, sans-serif; }
h1, h2 { padding-bottom: .3em; border-bottom: 1px solid var(--border); }
a { color: var(--link); }
pre, code { background: var(--code-bg); border-radius: 6px; font-family: ui-monospace, SFMono-Regular, Menlo, monospace; }
code { padding: .2em .4em; font-size: 85%; }
pre { padding: 16px; overflow: auto; }
pre code { padding: 0; background: none; }
blockquote { margin: 0; padding: 0 1em; color: var(--muted); border-left: .25em solid var(--border); }
table { border-collapse: collapse; }
th, td { padding: 6px 13px; border: 1px solid var(--border); }
hr { border: 0; border-top: 1px solid var(--border); }
img, svg { max-width: 100%; height: auto; }
</style>
</head>
<body>
{% endraw %}{{ content }}{% raw %}
</body>
</html>
{% endraw %}
