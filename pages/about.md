---
layout: page
title: About
description: 持续学习……
keywords: 好学长, haoxuezhang，何佳豪
comments: true
menu: 关于
permalink: /about/
---

在Coding中不断学习

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
