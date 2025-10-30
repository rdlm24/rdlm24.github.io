---
layout: page
permalink: /repositories/
title: repositorios
description: 
nav: true
nav_order: 4
---

## Repositorios Github

Aquí encontrarás algunos proyectos de desarrollo web que he creado, así como experimentos y herramientas relacionadas con IA y machine learning.

{% if site.data.repositories.github_users %}

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo_user.liquid username=site.github_username %}
</div>

{% endif %}

---

<div style="margin-top: 2rem;">
  <p style="color: #0E6E6E;">
    <i class="fab fa-github"></i> Visita mi perfil completo en 
    <a href="https://github.com/{{ site.github_username }}" target="_blank" style="font-weight: bold;">GitHub</a>
  </p>
</div>
