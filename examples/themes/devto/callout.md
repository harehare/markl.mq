> **{{ type | default("note") | capitalize }}**
> {{ body | trim | replace("\n", "\n> ") }}
