{% raw %}<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{% endraw %}{{ page[:title] }}{% raw %}</title>
<style>
:root { --bg: #ffffff; --fg: #2f3a46; --muted: #6b7a8a; --border: #e1e8ee; --code-bg: #edf2f7; --link: #3ea8ff; }
@media (prefers-color-scheme: dark) {
  :root { --bg: #1b1f27; --fg: #d6dde5; --muted: #8b98a8; --border: #323a46; --code-bg: #272d38; --link: #5eb6ff; }
}
body { max-width: 740px; margin: 0 auto; padding: 40px 16px; background: var(--bg); color: var(--fg);
  font: 16px/1.9 "Hiragino Kaku Gothic ProN", "Noto Sans JP", Meiryo, sans-serif; }
h1 { font-size: 1.9em; line-height: 1.4; }
h2 { margin-top: 2.4em; padding-bottom: .3em; border-bottom: 1px solid var(--border); }
a { color: var(--link); }
pre, code { background: var(--code-bg); border-radius: 4px; font-family: Menlo, Consolas, monospace; }
code { padding: .15em .35em; font-size: 90%; }
pre { padding: 1em; overflow: auto; line-height: 1.6; }
pre code { padding: 0; background: none; }
blockquote { margin: 1em 0; padding: 0 1em; color: var(--muted); border-left: 4px solid var(--border); }
div.message { margin: 1.5em 0; padding: 0 1em; border-radius: 4px; background: var(--code-bg); border-left: 4px solid var(--link); }
div.message.alert { border-left-color: #e5484d; }
img, svg { max-width: 100%; height: auto; }
</style>
</head>
<body>
{% endraw %}{{ content }}{% raw %}
</body>
</html>
{% endraw %}
