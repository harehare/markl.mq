> [!{{ type | default("note") | upper }}]
> {{ body | trim | replace("\n", "\n> ") }}
