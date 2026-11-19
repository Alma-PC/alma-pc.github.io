---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---

{% if author.googlescholar %}
  You can also find my articles on <u><a href="{{author.googlescholar}}">my Google Scholar profile</a>.</u>
{% endif %}

{% include base_path %}

You can also find my research records on my [ORCID profile](https://orcid.org/0009-0008-6360-0325).

{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
