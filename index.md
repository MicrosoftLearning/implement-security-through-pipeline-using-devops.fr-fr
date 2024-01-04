---
title: 'Exercices d’implémentation de la sécurité avec un pipeline à l’aide d’Azure DevOps '
permalink: index.html
layout: home
---

# Exercices d’implémentation de la sécurité avec un pipeline à l’aide d’Azure DevOps

Les exercices suivants sont conçus pour prendre en charge les modules sur [Implémenter la sécurité via un pipeline à l’aide de DevOps](https://learn.microsoft.com/training/paths/implement-security-through-pipeline-using-devops/).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
