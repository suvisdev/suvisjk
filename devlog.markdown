---
layout: default
title: 개발 로그
permalink: /devlog/
nav_order: 8
---

# 개발 진행 현황

---

## Mova — AI 영화 추천

{% assign mova_posts = site.posts | where_exp: "post", "post.categories contains 'mova'" %}
{% if mova_posts.size > 0 %}
{% for post in mova_posts %}
<div style="margin-bottom: 1.2rem; padding-bottom: 0.8rem; border-bottom: 1px solid #333;">
  <p style="margin:0; color:#999; font-size:0.85rem;">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 style="margin:0.3rem 0;"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  {% if post.categories.size > 0 %}
  <span style="font-size:0.8rem; color:#22c55e;">{{ post.categories | join: ", " }}</span>
  {% endif %}
</div>
{% endfor %}
{% else %}
<p style="color:#999;">아직 Mova 관련 로그가 없습니다.</p>
{% endif %}

---

## Gildle — 산책 경로 추천

{% assign gildle_posts = site.posts | where_exp: "post", "post.categories contains 'gildle'" %}
{% if gildle_posts.size > 0 %}
{% for post in gildle_posts %}
<div style="margin-bottom: 1.2rem; padding-bottom: 0.8rem; border-bottom: 1px solid #333;">
  <p style="margin:0; color:#999; font-size:0.85rem;">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 style="margin:0.3rem 0;"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  {% if post.categories.size > 0 %}
  <span style="font-size:0.8rem; color:#22c55e;">{{ post.categories | join: ", " }}</span>
  {% endif %}
</div>
{% endfor %}
{% else %}
<p style="color:#999;">아직 Gildle 관련 로그가 없습니다.</p>
{% endif %}

---

## 공통 (인프라 · 보안 · 기타)

{% assign other_posts = site.posts | where_exp: "post", "post.categories contains 'mova' or post.categories contains 'gildle'" %}
{% assign all_posts = site.posts %}
{% for post in all_posts %}
{% unless post.categories contains 'mova' or post.categories contains 'gildle' %}
<div style="margin-bottom: 1.2rem; padding-bottom: 0.8rem; border-bottom: 1px solid #333;">
  <p style="margin:0; color:#999; font-size:0.85rem;">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 style="margin:0.3rem 0;"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  {% if post.categories.size > 0 %}
  <span style="font-size:0.8rem; color:#22c55e;">{{ post.categories | join: ", " }}</span>
  {% endif %}
</div>
{% endunless %}
{% endfor %}
