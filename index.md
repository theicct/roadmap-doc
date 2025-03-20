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

{% comment %}The following block was written by Claude and handles the ordering of versions so that v1.10 appears after v1.9 rather than between v1.1 and v1.2.{% endcomment %}

{% assign pages = site.pages | where_exp: "page", "page.dir contains '/versions/' and page.title contains 'Roadmap v'" %}

{% comment %}First, create arrays to hold our pages with their version numbers{% endcomment %}
{% assign versioned_pages = "" | split: "" %}

{% for page in pages %}
  {% comment %}Extract version number from title (e.g., extract "1.5" from "Roadmap v1.5 Model Documentation"){% endcomment %}
  {% assign version_string = page.title | split: "Roadmap v" | last | split: " " | first %}
  
  {% comment %}Split into major and minor parts (e.g., "1" and "5"){% endcomment %}
  {% assign version_parts = version_string | split: "." %}
  {% assign major = version_parts[0] | plus: 0 %}
  {% assign minor = version_parts[1] | plus: 0 %}
  
  {% comment %}Compute a sortable number (major * 1000 + minor) to handle correct version ordering{% endcomment %}
  {% assign sort_key = major | times: 1000 | plus: minor %}
  
  {% comment %}Store the page with its sort key{% endcomment %}
  {% capture page_with_key %}{{ sort_key }}|||{{ page.url }}|||{{ page.title }}{% endcapture %}
  {% assign versioned_pages = versioned_pages | push: page_with_key %}
{% endfor %}

{% comment %}Sort the array by the computed sort keys{% endcomment %}
{% assign sorted_pages = versioned_pages | sort | reverse %}

{% comment %}Display the sorted list{% endcomment %}
{% for page_info in sorted_pages %}
  {% assign parts = page_info | split: "|||" %}
  <li><a class="page-link" href="{{ parts[1] | relative_url }}">{{ parts[2] | escape }}</a></li>
{% endfor %}
