---
layout: default
title: 개발 로그
permalink: /devlog/
nav_order: 8
---

# 개발 진행 현황

<style>
.devlog-tabs { display: flex; gap: 0.5rem; margin-bottom: 1.5rem; flex-wrap: wrap; }
.devlog-tab {
  padding: 0.5rem 1.2rem; border-radius: 2rem; cursor: pointer;
  font-size: 0.9rem; font-weight: 600; border: 1px solid #444;
  background: transparent; color: #ccc; transition: all 0.2s;
}
.devlog-tab:hover { border-color: #888; color: #fff; }
.devlog-tab.active { background: #7253ed; border-color: #7253ed; color: #fff; }
.devlog-panel { display: none; }
.devlog-panel.active { display: block; }
.devlog-entry {
  margin-bottom: 1.2rem; padding: 1rem 1.2rem;
  border-left: 3px solid #444; background: rgba(255,255,255,0.03);
  border-radius: 0 8px 8px 0;
}
.devlog-entry:hover { border-left-color: #7253ed; }
.devlog-date { font-size: 0.8rem; color: #999; margin: 0; }
.devlog-title { margin: 0.2rem 0 0.4rem; }
.devlog-title a { color: #e2e2e2; text-decoration: none; }
.devlog-title a:hover { color: #7253ed; }
.devlog-tags { display: flex; gap: 0.4rem; flex-wrap: wrap; }
.devlog-tag {
  font-size: 0.7rem; padding: 0.15rem 0.5rem;
  border-radius: 1rem; background: rgba(114,83,237,0.15); color: #a78bfa;
}
.devlog-summary { font-size: 0.85rem; color: #bbb; margin-top: 0.5rem; line-height: 1.5; }
</style>

<div class="devlog-tabs">
  <button class="devlog-tab active" onclick="showTab('all')">전체</button>
  <button class="devlog-tab" onclick="showTab('mova')">Mova</button>
  <button class="devlog-tab" onclick="showTab('gildle')">Gildle</button>
  <button class="devlog-tab" onclick="showTab('infra')">인프라 · 보안</button>
</div>

<div id="panel-all" class="devlog-panel active">

{% for post in site.posts %}
<div class="devlog-entry">
  <p class="devlog-date">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 class="devlog-title"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <div class="devlog-tags">
    {% for cat in post.categories %}<span class="devlog-tag">{{ cat }}</span>{% endfor %}
  </div>
  {% if post.excerpt %}<div class="devlog-summary">{{ post.excerpt | strip_html | truncate: 120 }}</div>{% endif %}
</div>
{% endfor %}

</div>

<div id="panel-mova" class="devlog-panel">

{% assign mova_posts = site.posts | where_exp: "post", "post.categories contains 'mova'" %}
{% if mova_posts.size > 0 %}
<p style="color:#a78bfa; font-size:0.85rem; margin-bottom:1rem;">총 {{ mova_posts.size }}건의 Mova 개발 로그</p>
{% for post in mova_posts %}
<div class="devlog-entry">
  <p class="devlog-date">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 class="devlog-title"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <div class="devlog-tags">
    {% for cat in post.categories %}<span class="devlog-tag">{{ cat }}</span>{% endfor %}
  </div>
  {% if post.excerpt %}<div class="devlog-summary">{{ post.excerpt | strip_html | truncate: 120 }}</div>{% endif %}
</div>
{% endfor %}
{% else %}
<p style="color:#999;">아직 Mova 관련 로그가 없습니다.</p>
{% endif %}

</div>

<div id="panel-gildle" class="devlog-panel">

{% assign gildle_posts = site.posts | where_exp: "post", "post.categories contains 'gildle'" %}
{% if gildle_posts.size > 0 %}
<p style="color:#a78bfa; font-size:0.85rem; margin-bottom:1rem;">총 {{ gildle_posts.size }}건의 Gildle 개발 로그</p>
{% for post in gildle_posts %}
<div class="devlog-entry">
  <p class="devlog-date">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 class="devlog-title"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <div class="devlog-tags">
    {% for cat in post.categories %}<span class="devlog-tag">{{ cat }}</span>{% endfor %}
  </div>
  {% if post.excerpt %}<div class="devlog-summary">{{ post.excerpt | strip_html | truncate: 120 }}</div>{% endif %}
</div>
{% endfor %}
{% else %}
<p style="color:#999;">아직 Gildle 관련 로그가 없습니다.</p>
{% endif %}

</div>

<div id="panel-infra" class="devlog-panel">

{% for post in site.posts %}
{% unless post.categories contains 'mova' or post.categories contains 'gildle' %}
<div class="devlog-entry">
  <p class="devlog-date">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
  <h3 class="devlog-title"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <div class="devlog-tags">
    {% for cat in post.categories %}<span class="devlog-tag">{{ cat }}</span>{% endfor %}
  </div>
  {% if post.excerpt %}<div class="devlog-summary">{{ post.excerpt | strip_html | truncate: 120 }}</div>{% endif %}
</div>
{% endunless %}
{% endfor %}

</div>

<script>
function showTab(name) {
  document.querySelectorAll('.devlog-panel').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.devlog-tab').forEach(t => t.classList.remove('active'));
  document.getElementById('panel-' + name).classList.add('active');
  event.target.classList.add('active');
}
</script>
