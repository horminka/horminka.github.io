---
layout: listas
title: Conferencias
ref: conferencias
permalink: /conferencias/
lang: Español
---

# Conferencias

{% for conferencia in site.data.conferencias %}
  * [{{ conferencia.nombre }}](#{{ conferencia.nombre | replace:'á','' | replace:'é','' | replace:'í','' | replace:'ó','' | replace:'ú','' | slugify }})
{% endfor %}

{% for conferencia in site.data.conferencias %}
## {{ conferencia.nombre }}
{{ conferencia.descripcion }}

### Objetivos
{% for objetivo in conferencia.objetivos %}
  * {{ objetivo }}
{% endfor %}

### Dirigido a
{{ conferencia.dirigido-a }}

### Contenido
{% for contenido in conferencia.contenidos %}
  * {{ contenido }}
{% endfor %}

### Duración
{{ conferencia.duracion }}
{% endfor %}
