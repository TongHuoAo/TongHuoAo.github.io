---
layout: page
title: About
permalink: /about/
weight: 3
---

# **About Me**

Hi I am **{{ site.author.name }}** :smile:,<br>


{% include elements/button.html link="https://drive.google.com/file/d/1c2nbXzyJpVmy30VTKHNhx0XYi31kjya1/view?usp=sharing" text="backend SDE resume" %}
{% include elements/button.html link="https://drive.google.com/file/d/1uMT7edEXa0A8qtgke6p45MZtgmmr388g/view?usp=sharing" text="SDE in Graphics resume"%}

<div class="row">
{% include about/skills.html title="Programming Skills" source=site.data.programming-skills %}
{% include about/skills.html title="" source=site.data.other-skills %}
</div>

## Experience
<div class="row">
{% include about/timeline.html %}
</div>