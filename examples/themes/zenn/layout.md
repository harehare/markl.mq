{%- extends "default/layout" -%}
{%- block front_matter -%}
{{ {title: page[:title], emoji: default("📝", page[:emoji]), type: "tech", topics: page[:tags], published: default(false, page[:published])} | frontmatter() }}
{%- endblock -%}
{%- block title %}{% endblock -%}
{%- block meta %}{% endblock -%}
{%- block toc %}{% endblock -%}
{%- block footer %}{% endblock -%}
