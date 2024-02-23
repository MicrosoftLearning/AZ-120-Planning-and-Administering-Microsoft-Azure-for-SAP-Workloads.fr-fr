---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

# Répertoire de contenu : AZ-120 : Planification et administration des charges de travail Microsoft Azure pour SAP

Les fichiers des labos requis sont disponibles en [téléchargement ici](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads /archive/master.zip)

Les liens hypertexte vers chaque exercice et démonstration de labo sont répertoriés ci-dessous.

## Laboratoires

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/'" %}
| Module | Laboratoire |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
