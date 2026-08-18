---
layout: page
title: Projetos
lang: pt-br
permalink: /pt-br/projects/
---

{%- assign localized_projects = site.projects | where: "lang", "pt-br" -%}
<ul class="project-list">
{%- for project in localized_projects -%}
  <li>
    <a href="{{ project.url | relative_url }}">{{ project.title }}</a>
    {%- if project.summary %} — {{ project.summary }}{% endif %}
  </li>
{%- endfor -%}
</ul>
