{# Jinja template for GDC dictionary render in markdown #}
{# input vars : obj - the dictionary json object #}
{#              gloss - dict mapping term names to term objs #}
{#              has_terms - if true, create terms table #}

{% macro link_to(nm) -%}
[{{nm}}](#{{nm}})
{%- endmacro %}
{% macro anchor_on(nm) -%}
<a name="{{nm}}"></a>
{%- endmacro %}
{% macro titleize(nm) -%}
 {% set S = nm.split("_") %}
 {% set t = S[0]|capitalize %}
 {% for s in S %}
 {% set t = t + ' ' + s|capitalize %}
 {% endfor %}
{{t}}
{%- endmacro %}
## _Entry_: {{ obj.title }} ##
---
{% if gloss.get_definition(obj.id) %}
_Definition_: {{ gloss.get_definition(obj.id) }}
{% elif obj.description is defined and obj.description != 'TBD'%}
_Description_: {{ obj.description }}
{% else %}
_Description_: _*Coming Soon*_
{% endif %}
_Category_: `{{obj.category}}`
{% if gloss.get_cde_info(obj.id) is defined %}
   {% set cdeinfo = gloss.get_cde_info(obj.id) %}
   {% if cdeinfo.collection_url %}
_CDE_: [{{cdeinfo.cde_id}} ({{cdeinfo.collection_name}})]({{cdeinfo.collection_url}})
   {% elif cdeinfo.cde_id %}
_CDE_: {{cdeinfo.cde_id}}
   {% endif %}
{% endif %}
---

{% if obj.properties is defined %}
### Properties ###
| Property | Acceptable Type or Values | Required |
| --- | --- | --- |
{% for prop, propv in (obj.properties|dictsort) %}
   {% if prop=='type' %}{% continue %}{% endif %}
   {% if propv.type is defined %}
| {{ link_to(prop) if gloss.get_term(prop) else prop }} | _{{propv.type}}_ | {{ 'YES' if prop in obj.required else 'no' }} |
   {% elif propv.enum is defined %}
| {{ link_to(prop) if gloss.get_term(prop) else prop }} | "{{ propv.enum|join('", "') }}" | {{ 'YES' if prop in obj.required else 'no' }} |
   {% endif %}
{% endfor %}
{% endif %}

{% if obj.links|length %}
### Links ###
| Link to Entry | Relationship | Required |
| --- | --- | --- |
{% for l in obj.links %}
{% if 'subgroup' in l %}
 {% for link in l.subgroup %}
 | [{{titleize(link.target_type)}}](/gdc-dictionary/{{titleize(link.target_type)}}) | {{ 'A' if obj.title[0] not in 'AEIOUaeious' else 'An' }} {{obj.title}} record {{link.label}} {{ 'a' if link.target_type[0] not in 'AEIOUaeiou' else 'an' }} {{link.target_type}} | {{ 'YES' if link.required else 'no' }} |
 {% endfor %}
 {% else %}
  | [{{titleize(l.target_type)}}](/gdc-dictionary/{{titleize(l.target_type)}}) | {{ 'A' if obj.title[0] not in 'AEIOUaeiou' else 'An' }} {{obj.title}} record {{l.label}} {{ 'a' if l.target_type[0] not in 'AEIOUaeiou' else 'an' }} {{l.target_type}} | {{ 'YES' if l.required else 'no' }} |
 {% endif %}
{% endfor %}
{% endif %}

{% if has_terms %}
### Terms ###
| Term | Definition | Collection | CDE |
| --- | --- | --- | --- |
{% for prop,propv in obj.properties|dictsort %}
{% if gloss.get_term(prop) %}
   {% if gloss.get_cde_info(prop).collection_url %}
| {{prop}} | {{anchor_on(prop)}}{{gloss.get_definition(prop)}} | {{gloss.get_cde_info(prop).collection_name}} |[{{gloss.get_cde_info(prop).cde_id}}]({{gloss.get_cde_info(prop).collection_url}})|
   {% else %}
| {{prop}} | {{anchor_on(prop)}}{{gloss.get_definition(prop)}} | {{gloss.get_cde_info(prop).collection_name}} |{{gloss.get_cde_info(prop).cde_id}}|
   {% endif %}
{% endif %}
{% endfor %}
{% endif %}