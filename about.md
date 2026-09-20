---
layout: page
title: About
permalink: /about/
---
Problem solver by habit, computer engineer by training. I like learning whatever a problem actually needs, and I keep small tools around so I can see an idea working.

This site is a project gallery first. The write-ups are short on purpose: what it is, why it exists, and a link to the code.

## Background

<ul class="timeline">
  <li>
    <time datetime="2018">2018</time>
    <strong>Middleware engineer</strong> at <a href="https://www.emerio.com/">PT Emerio Indonesia</a>.
  </li>
  <li>
    <time datetime="2015">2015</time>
    <strong>University internships.</strong> Telkom University placed me at <a href="https://www.bpjs-kesehatan.go.id/">BPJS Kesehatan</a> in Banda Aceh, then I spent a second internship at a small beauty clinic with no IT team — POS updates, an attendance machine, and whatever else was broken that week.
  </li>
  <li>
    <time datetime="2013/2017">2013–2017</time>
    <strong>B.Eng. Computer Engineering</strong> at <a href="https://telkomuniversity.ac.id/">Telkom University</a> (GPA 3.26 / 4.00). Systems, programming, networks, with more time on software than anything else. The final project was sign detection and digit recognition on a Raspberry Pi — that write-up is under <a href="{{ '/work/tugas-akhir/' | relative_url }}">Work</a>.
  </li>
</ul>

## Contact

<p>
{% for social in site.social %}
<a href="{{ social.url }}">{{ social.title }}</a>{% unless forloop.last %} · {% endunless %}
{% endfor %}
</p>
