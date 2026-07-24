---
title: "下载"
layout: single
classes: wide top-page download-page
lang: zh
lang-ref: download
permalink: /zh/download/
---

{% assign current = site.data.releases.current %}

<section class="download-hero">
  <p class="download-kicker">ACNN 软件包</p>
  <h2>下载 ACNN-CSP，并查看版本更新记录。</h2>
  <p>ACNN-CSP 目前直接通过官网提供下载，发布形式为 Linux x86-64 本地 Conda channel 压缩包。该压缩包包含 ACNN 可执行程序、ACNN-Relax、晶体结构预测工具、工作流工具和声明的运行时依赖。</p>
  <div class="download-actions">
    <a class="download-button" href="{{ current.download_url | relative_url }}">下载 ACNN-CSP {{ current.version }}</a>
    <a class="download-link" href="{{ current.checksum_url | relative_url }}">SHA256 校验文件</a>
    <a class="download-link" href="{{ '/zh/installation/' | relative_url }}">查看安装方法</a>
  </div>
</section>

<section class="release-current">
  <div class="release-heading">
    <p class="download-kicker">当前版本</p>
    <h2>{{ current.version }}</h2>
    {% if current.date %}
      <p class="release-date">{{ current.date }}</p>
    {% endif %}
    <p class="release-status">{{ current.status }}</p>
    <p class="release-status">压缩包大小：{{ current.size_bytes }} bytes ({{ current.size_mib }} MiB)</p>
  </div>

  <div class="release-notes-grid">
    <section>
      <h3>新功能</h3>
      <ul>
        {% for item in current.new_features_zh %}
          <li>{{ item }}</li>
        {% endfor %}
      </ul>
    </section>
    <section>
      <h3>问题修复</h3>
      <ul>
        {% for item in current.bug_fixes_zh %}
          <li>{{ item }}</li>
        {% endfor %}
      </ul>
    </section>
    <section>
      <h3>兼容性</h3>
      <ul>
        {% for item in current.compatibility_zh %}
          <li>{{ item }}</li>
        {% endfor %}
      </ul>
    </section>
  </div>
</section>

<details class="release-history">
  <summary>
    <span class="download-kicker">版本历史</span>
    <span class="release-history-title">历史版本</span>
  </summary>

  {% for release in site.data.releases.history %}
    <article class="release-history-item">
      <div>
        <h3>{{ release.version }}</h3>
        {% if release.date %}
          <p class="release-date">{{ release.date }}</p>
        {% endif %}
      </div>
      <div>
        <p class="release-status">{{ release.status }}</p>
        <ul>
          {% for note in release.notes_zh %}
            <li>{{ note }}</li>
          {% endfor %}
        </ul>
      </div>
    </article>
  {% endfor %}
</details>
