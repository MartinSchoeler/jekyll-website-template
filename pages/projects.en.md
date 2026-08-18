---
layout: page
title: Projects
lang: en
permalink: /projects/
---

{%- assign localized_projects = site.projects | where: "lang", "en" -%}
<ul class="project-list">
{%- for project in localized_projects -%}
  <li>
    <a href="{{ project.url | relative_url }}">{{ project.title }}</a>
    {%- if project.summary %} — {{ project.summary }}{% endif %}
  </li>
{%- endfor -%}
</ul>
