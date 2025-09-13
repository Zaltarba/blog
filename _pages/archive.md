---
layout: page
permalink: /archive/
title: Posts Archive
---

<style>
  /* Timeline container */
  #timeline {
    position: relative;
    max-width: 700px;
    margin: 0 auto;
    padding-left: 40px;
  }

  /* The vertical timeline line */
  #timeline::before {
    content: '';
    position: absolute;
    left: 20px;
    top: 0;
    bottom: 0;
    width: 4px;
    background-color: #ddd;
  }

  /* Timeline item */
  .timeline-item {
    position: relative;
    margin-bottom: 30px;
    padding-left: 30px;
  }

  /* Circle marker */
  .timeline-item::before {
    content: '';
    position: absolute;
    left: 8px;
    top: 5px;
    width: 16px;
    height: 16px;
    background-color: #4CAF50;
    border-radius: 50%;
    border: 3px solid white;
  }

  /* Year heading */
  .timeline-year {
    margin-top: 40px;
    font-size: 1.5em;
    font-weight: bold;
    border-bottom: 2px solid #4CAF50;
    padding-bottom: 5px;
  }

  /* Month heading */
  .timeline-month {
    font-size: 1.2em;
    font-weight: 600;
    color: #555;
    margin-top: 10px;
    margin-bottom: 8px;
  }

  /* Post link */
  .post-link {
    font-weight: bold;
    color: #2c3e50;
    text-decoration: none;
  }
  .post-link:hover {
    text-decoration: underline;
  }

  /* Post date */
  .post-date {
    font-size: 0.85em;
    color: #888;
    margin-left: 5px;
  }
</style>

<div id="timeline">
  {% assign current_year = '' %}
  {% assign current_month = '' %}

  {% for post in site.posts %}
    {% assign post_year = post.date | date: '%Y' %}
    {% assign post_month = post.date | date: '%B' %}

    {% if post_year != current_year %}
      <h2 class="timeline-year">{{ post_year }}</h2>
      {% assign current_year = post_year %}
      {% assign current_month = '' %}
    {% endif %}

    {% if post_month != current_month %}
      <h3 class="timeline-month">{{ post_month }}</h3>
      {% assign current_month = post_month %}
    {% endif %}

    <div class="timeline-item">
      <a class="post-link" href="{{ site.baseurl }}{{ post.url }}">
        {% if post.title and post.title != "" %}
          {{ post.title }}
        {% else %}
          {{ post.excerpt | strip_html | truncate: 60 }}
        {% endif %}
      </a>
      <span class="post-date">{{ post.date | date: "%-d %b %Y" }}</span>
    </div>
  {% endfor %}
</div>
