---
layout: listas
title: Talleres
ref: talleres
permalink: /talleres/
lang: Español
---

# Talleres

{% for taller in site.data.talleres %}
  * [{{ taller.nombre }}](#{{ taller.nombre | replace:'á','' | replace:'é','' | replace:'í','' | replace:'ó','' | replace:'ú','' | slugify }})
{% endfor %}

{% for taller in site.data.talleres %}
## {{ taller.nombre }}
{{ taller.descripcion }}

### Objetivos
{% for objetivo in taller.objetivos %}
  * {{ objetivo }}
{% endfor %}

### Dirigido a
{{ taller.dirigido-a }}

### Duración
{{ taller.duracion }}

### Metodología
{{ taller.metodologia }}

### Prerrequisitos
{{ taller.prerrequisitos }}

### Materiales
{% for material in taller.materiales %}
  * {{ material }}
{% endfor %}

### Contenido
{% for contenido in taller.contenidos %}
  * {{ contenido }}
{% endfor %}
{% endfor %}
