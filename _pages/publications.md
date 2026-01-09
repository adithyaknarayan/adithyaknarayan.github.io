---
layout: page
permalink: /publications/
title: publications
description: Selected publications and a complete list in reverse chronological order.
nav: true
nav_order: 2
---

<!-- _pages/publications.md -->
<div class="publications">

<h2>selected</h2>
{% bibliography --group_by none --query @*[selected=true]* %}

<h2>all</h2>
{% bibliography %}

</div>
