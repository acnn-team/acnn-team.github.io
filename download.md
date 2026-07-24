---
title: "Download"
layout: single
classes: wide top-page download-page
lang: en
lang-ref: download
permalink: /download/
---

{% assign current = site.data.releases.current %}

<section class="download-hero">
  <p class="download-kicker">ACNN Package</p>
  <h2>Download ACNN-CSP and review release updates.</h2>
  <p>ACNN-CSP is currently distributed directly from this website as a Linux x86-64 local Conda channel archive. The archive installs ACNN executables, ACNN-Relax, crystal-structure prediction utilities, workflow tools, and declared runtime dependencies.</p>
  <div class="download-actions">
    <a class="download-button" href="{{ current.download_url | relative_url }}">{{ current.download_label }}</a>
    <a class="download-link" href="{{ current.checksum_url | relative_url }}">{{ current.checksum_label }}</a>
    <a class="download-link" href="{{ '/getting-started/installation/' | relative_url }}">Installation guide</a>
  </div>
</section>

<section class="release-current">
  <div class="release-heading">
    <p class="download-kicker">Current Release</p>
    <h2>{{ current.version }}</h2>
    {% if current.date %}
      <p class="release-date">{{ current.date }}</p>
    {% endif %}
    <p class="release-status">{{ current.status }}</p>
    <p class="release-status">Archive size: {{ current.size_bytes }} bytes ({{ current.size_mib }} MiB)</p>
  </div>

  <div class="release-notes-grid">
    <section>
      <h3>New Features</h3>
      <ul>
        {% for item in current.new_features %}
          <li>{{ item }}</li>
        {% endfor %}
      </ul>
    </section>
    <section>
      <h3>Bug Fixes</h3>
      <ul>
        {% for item in current.bug_fixes %}
          <li>{{ item }}</li>
        {% endfor %}
      </ul>
    </section>
    <section>
      <h3>Compatibility</h3>
      <ul>
        {% for item in current.compatibility %}
          <li>{{ item }}</li>
        {% endfor %}
      </ul>
    </section>
  </div>
</section>

<details class="release-history">
  <summary>
    <span class="download-kicker">Version History</span>
    <span class="release-history-title">Previous releases</span>
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
          {% for note in release.notes %}
            <li>{{ note }}</li>
          {% endfor %}
        </ul>
      </div>
    </article>
  {% endfor %}
</details>
