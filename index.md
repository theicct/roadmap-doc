---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: Roadmap Model Documentation
---

The ICCT’s Roadmap model is a global transportation emissions model covering all on-road vehicle activity in over 190 countries. The Roadmap model is intended to help policymakers worldwide to identify and understand trends in the transportation sector, assess emission impacts of different policy options, and frame plans to effectively reduce emissions of both greenhouse gases (GHGs) and local air pollutants. It is designed to allow transparent, customizable estimation of transportation emissions for a broad range of policy cases.

Roadmap was first developed in 2019 by Caleb Braun, Lingzhi Jin, and Josh Miller. 

## Versions

Roadmap is under continuing development. Documentation of all versions since v1.5 can be found here.

{% assign pages = site.pages | sort: "title" | reverse %}
{% for page in pages %}
{% if page.dir contains '/versions/' and page.title contains 'Roadmap v'%}
<li><a class="page-link" href="{{ page.url | relative_url }}">{{ page.title | escape }}</a></li>
{% endif %}
{% endfor %}
