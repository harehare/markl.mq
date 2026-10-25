{%- extends "default/layout" -%}
{%- block front_matter -%}
{% set tags = join(default([], page[:tags]), ", ") -%}
{{ {title: page[:title], published: default(false, page[:published]), description: page[:description], tags: default(None, tags), cover_image: page[:cover_image]} | frontmatter() }}{{ "\n" }}
{%- endblock -%}
{%- block title %}{% endblock -%}
{%- block meta %}{% endblock -%}
