---
layout: page
permalink: /categories/
title: Categories
---

<div id="categories">
  {% for category in site.categories %}
    {% assign category_name = category | first %}
    <div class="category-group">
      
      <!-- Anchor for each category -->
      <a id="{{ category_name | slugize }}"></a>
      
      <!-- Category Header -->
      <h3 class="category-head">{{ category_name }}</h3>
      
      <!-- Category Posts -->
      <ul class="category-post-list">
        {% for post in category.last %}
          <li class="category-post-item">
            <a href="{{ post.url | prepend: site.baseurl }}">
              {% if post.title %}
                {{ post.title }}
              {% else %}
                {{ post.excerpt | strip_html }}
              {% endif %}
            </a>
          </li>
        {% endfor %}
      </ul>
    </div>
  {% endfor %}
</div>
