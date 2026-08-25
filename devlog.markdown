---
layout: default
title: 개발 로그
permalink: /devlog/
nav_order: 8
---

# 개발 진행 현황

{% if site.posts.size > 0 %}
{% for post in site.posts %}
<div style="margin-bottom: 1.5rem; padding-bottom: 1rem; border-bottom: 1px solid #eee;">
  <p style="margin:0; color:#999; font-size:0.85rem;">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 style="margin:0.3rem 0;"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  {% if post.categories.size > 0 %}
  <span style="font-size:0.8rem; color:#22c55e;">{{ post.categories | join: ", " }}</span>
  {% endif %}
</div>
{% endfor %}
{% else %}
<p style="color:#999; padding:2rem 0;">아직 작성된 개발 로그가 없습니다. <code>_posts/</code> 디렉터리에 포스트를 추가하세요.</p>
{% endif %}
