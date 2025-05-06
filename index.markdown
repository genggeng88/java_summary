---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

# layout: home

layout: default
title: Knowledge Summary
---

# Welcome to My Knowledge Repository

Below is a list of topics I want to share:

{% for knowledge in site.knowledge %}
- **{{ knowledge.topic }}**: [{{ knowledge.title }}]({{ site.baseurl }}{{ knowledge.url }})
{% endfor %}