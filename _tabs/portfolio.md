---
title: Portfolio
icon: fas fa-briefcase
order: 2
---

Below is a selection of my data, automation, and cloud projects.
Each card links to a detailed writeup.

**Quick filters:**  
[All](/portfolio/) · [Microsoft Fabric projects](/categories/fabric/) · [Automation projects](/categories/automation/)

**Quick filters:**  
[All]({{ '/portfolio/' | relative_url }}) ·
[Microsoft Fabric projects]({{ '/categories/fabric/' | relative_url }}) ·
[Automation projects]({{ '/categories/automation/' | relative_url }})


<div class="portfolio-grid">
  {% assign projects = site.posts
     | where_exp: "post", "post.categories contains 'portfolio'"
     | sort: "date" | reverse %}

  {% for project in projects %}
    <article class="project-card">
      <div class="project-card-inner">
        {% if project.icon %}
          <div class="project-icon">
            <img src="{{ project.icon | relative_url }}" alt="">
          </div>
        {% endif %}

        <h2 class="project-title">
          <a href="{{ project.url | relative_url }}">{{ project.title }}</a>
        </h2>

        <p class="project-status">
          {{ project.status | default: "Not started" }}
        </p>

        <p class="project-tags">
          {% if project.categories[1] %}
            <span class="project-pill">
              {{ project.categories[1] | capitalize }}
            </span>
          {% endif %}

          {% for tag in project.tags %}
            <span class="project-pill">{{ tag }}</span>
          {% endfor %}
        </p>
      </div>
    </article>
  {% endfor %}
</div>
