---
layout: page
title: About
description: 你只管努力，剩下的交给天意
keywords:  liusCoding, liushuai
comments: true
menu: 关于
permalink: /about/
---

我是刘帅，湖南某渣渣二本计算机科学与技术毕业，现于深圳某坑爹公司从事Java开发。

仰慕「优雅编码的艺术」。

坚信熟能生巧，努力改变人生。

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
