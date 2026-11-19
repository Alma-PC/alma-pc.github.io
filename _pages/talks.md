---
layout: archive
title: "Talks"
permalink: /talks/
author_profile: true
redirect_from:
  - /talks
---

{% include base_path %}

Invited seminars and conference presentations.

{% for post in site.talks reversed %}
  {% include archive-single.html %}
{% endfor %}
