---
layout: default
title: Home
---

# Intro
Questo sito non ha la pretesa di essere dettagliato o esaustivo.  
Piuttosto vuole essere un insieme di buone norme e consigli utili per imparare a fare peering.  

---

# Argomenti
{% for topic in site.topics %}
  {% assign chunks = topic.id | split: "/" %}
  <h2 id="topic_{{ chunks.last }}"><a href="{{ topic.url }}">{{ topic.title }}</a></h2>
  {{ topic.subtitle }}
{% endfor %}

---

# Contribuisci
Maggiori informazioni su [{{site.github.repo}}]({{site.github.repo}}).
