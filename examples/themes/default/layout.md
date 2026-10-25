{%- block front_matter %}{% endblock -%}
{%- block title %}# {{ page[:title] }}

{% endblock -%}
{%- block meta -%}
*{% if page[:date] %}{{ page[:date] | date("%B %d, %Y") }} · {% endif %}{{ content | reading_time }} min read*

{% endblock -%}
{%- if page[:toc] != false -%}
{% block toc -%}
## Contents

{{ content | toc(2, 3) }}

{% endblock -%}
{%- endif -%}
{%- block body %}{{ content }}{% endblock %}
{%- block footer %}
---

Written with [markl.mq](https://github.com/harehare/markl.mq).
{% endblock -%}
