---
layout: page
title: About
permalink: /about/
---
Problem solver by habit, computer engineer by training. I like learning whatever a problem actually needs, and I keep small tools around so I can see an idea working.

Most of the day job is backend: services, APIs, databases, and glue between systems that were never meant to talk to each other. I will take time on a clean design when the calendar allows, and I will ship the smaller fix when it does not. Python is the default at work. Go is what I reach for when I want one binary.

This site is a project gallery first. The write-ups are short on purpose: what it is, why it exists, and a link to the code. Jobs are in the list below. The tools are under [Work]({{ '/work/' | relative_url }}).

## Background

<ul class="timeline">
  <li>
    <time datetime="2022">2022–2026</time>
    <strong>Backend</strong> at PT Teknologi Integrasi Optima (TekIno). Django and Flask systems, integrations into larger stacks, and client-facing work — including a WhatsApp assistant on their service.
  </li>
  <li>
    <time datetime="2022-03">Mar–Aug 2022</time>
    <strong>Freelance backend</strong> for the founder I later joined at TekIno. Python between mismatched systems, and authorized testing on a cellular-network security setup.
  </li>
  <li>
    <time datetime="2019">2019–2022</time>
    <strong>Backend</strong> at PT Mplus Software, on-site at Mobik. Servers, attack-detection software, Django for internal tools, and pieces of firewall code in C and Go.
  </li>
  <li>
    <time datetime="2018">2018–2019</time>
    <strong>Middleware</strong> at <a href="https://www.emerio.com/">PT Emerio Indonesia</a>. API gateways and portals, SOAP services turned into REST, and the usual test and deploy support.
  </li>
  <li>
    <time datetime="2016">2016</time>
    <strong>IT intern</strong> at a small beauty clinic (CV Cahaya Estetika) with no IT team. Network, servers, an attendance machine, ERP — and the queue system that became <a href="{{ '/work/queuemachine/' | relative_url }}">Queue Machine</a>. I still touch it when I visit.
  </li>
  <li>
    <time datetime="2015">2015</time>
    <strong>University internship</strong> at <a href="https://www.bpjs-kesehatan.go.id/">BPJS Kesehatan</a> in Banda Aceh, through Telkom University.
  </li>
  <li>
    <time datetime="2013">2013–2017</time>
    <strong>B.Eng. Computer Engineering</strong> at <a href="https://telkomuniversity.ac.id/">Telkom University</a> (GPA 3.26 / 4.00). Systems, programming, networks, with more time on software than anything else. The final project was sign detection and digit recognition on a Raspberry Pi — that write-up is under <a href="{{ '/work/tugas-akhir/' | relative_url }}">Work</a>.
  </li>
</ul>

## Contact

<p>
{% for social in site.social %}
<a href="{{ social.url }}">{{ social.title }}</a>{% unless forloop.last %} · {% endunless %}
{% endfor %}
</p>
