---
layout: default
title: Home
---

<style>
  .container {
    display: flex;
  }
  .main-content {
    flex: 3;
  }
  .sidebar {
    flex: 1;
    margin-right: 20px;
  }
</style>

<div class="container">
  <div class="sidebar">
    <h2>Open Source Contributions</h2>
    <ul>
      <li><a href="https://github.com/SigmaHQ/sigma/pull/4936">SigmaHQ Pull Request #4936</a></li>
      <li><a href="https://wtfbins.wtf/bin/32">WTFBins Bin 32</a></li>
    </ul>
  </div>

  <div class="main-content">
    <h1>Welcome</h1>

    - **Detection**
    - **Hunting**
    - **Malware**
    - **DFIR**

    security!

    <ul>
      {% for post in site.posts %}
        <li>
          <a href="{{ post.url }}">{{ post.title }}</a>
          <span>{{ post.date | date: "%B %d, %Y" }}</span>
        </li>
      {% endfor %}
    </ul>
  </div>
</div>

