---
title: Implement security through a pipeline using Azure DevOps exercises
permalink: index.html
layout: home
---

# Implement security through a pipeline using Azure DevOps exercises

The following exercises are designed to support the modules on [Implement security through a pipeline using DevOps](https://learn.microsoft.com/training/paths/implement-security-through-pipeline-using-devops/).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
{% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})
{% endfor %}
