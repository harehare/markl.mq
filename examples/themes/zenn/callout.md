{% if html -%}
<div class="message{% if type == "warning" || type == "caution" %} alert{% endif %}">

{{ body | trim }}

</div>
{%- else -%}
:::message{% if type == "warning" || type == "caution" %} alert{% endif %}
{{ body | trim }}
:::
{%- endif %}
