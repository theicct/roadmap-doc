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

{% comment %}The following block was written by Claude and handles the ordering of versions so that v2.10 appears after v2.9 rather than between v2.1 and v2.2.{% endcomment %}

{% assign roadmap_pages = site.pages | where_exp: "page", "page.dir contains '/versions/' and page.title contains 'Roadmap v'" %}

{% comment %}Group pages by major version{% endcomment %}
{% assign major_versions = "" | split: "" %}
{% for page in roadmap_pages %}
  {% assign version_text = page.title | split: "Roadmap v" | last | split: " " | first %}
  {% assign major = version_text | split: "." | first %}
  {% unless major_versions contains major %}
    {% assign major_versions = major_versions | push: major %}
  {% endunless %}
{% endfor %}

{% comment %}Sort major versions in ascending order{% endcomment %}
{% assign major_versions = major_versions | sort | reverse %}

{% comment %}Process each major version{% endcomment %}
{% for major in major_versions %}
  {% comment %}Get all pages for this major version{% endcomment %}
  {% assign major_pages = "" | split: "" %}
  {% for page in roadmap_pages %}
    {% assign version_text = page.title | split: "Roadmap v" | last | split: " " | first %}
    {% assign page_major = version_text | split: "." | first %}
    {% if page_major == major %}
      {% assign major_pages = major_pages | push: page %}
    {% endif %}
  {% endfor %}

  {% comment %}Separate pages with single-digit and multi-digit minor versions{% endcomment %}
  {% assign single_digit_pages = "" | split: "" %}
  {% assign multi_digit_pages = "" | split: "" %}

  {% for page in major_pages %}
    {% assign version_text = page.title | split: "Roadmap v" | last | split: " " | first %}
    {% assign minor = version_text | split: "." | last %}
    {% if minor.size == 1 %}
      {% assign single_digit_pages = single_digit_pages | push: page %}
    {% else %}
      {% assign multi_digit_pages = multi_digit_pages | push: page %}
    {% endif %}
  {% endfor %}

  {% comment %}Sort and display multi-digit versions first{% endcomment %}
  {% assign multi_digit_sorted = multi_digit_pages | sort: "title" | reverse %}
  {% for page in multi_digit_sorted %}
  <li><a class="page-link" href="{{ page.url | relative_url }}">{{ page.title | escape }}</a></li>
  {% endfor %}

  {% comment %}Sort and display single-digit versions second{% endcomment %}
  {% assign single_digit_sorted = single_digit_pages | sort: "title" | reverse %}
  {% for page in single_digit_sorted %}
  <li><a class="page-link" href="{{ page.url | relative_url }}">{{ page.title | escape }}</a></li>
  {% endfor %}
{% endfor %}
