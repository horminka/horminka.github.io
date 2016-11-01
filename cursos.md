---
layout: listas
title: Cursos
ref: cursos
permalink: /cursos/
lang: Español
---

# Cursos

{% for curso in site.data.cursos %}
  * [{{ curso.nombre }}](#{{ curso.nombre | replace:'á','' | replace:'é','' | replace:'í','' | replace:'ó','' | replace:'ú','' | slugify }})
{% endfor %}

{% for curso in site.data.cursos %}
## {{ curso.nombre }}
{{ curso.descripcion }}

### Objetivos
{% for objetivo in curso.objetivos %}
  * {{ objetivo }}
{% endfor %}

### Dirigido a
{{ curso.dirigido-a }}

### Duración
{{ curso.duracion }}

### Metodología
{{ curso.metodologia }}

### Prerrequisitos
{{ curso.prerrequisitos }}

### Materiales
{% for material in curso.materiales %}
  * {{ material }}
{% endfor %}

### Contenido
{% for contenido in curso.contenidos %}
  * {{ contenido }}
{% endfor %}
{% endfor %}
