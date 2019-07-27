---
layout: page
title: About
description: 个人博客，属于我的技术小世界！
keywords:  liusCoding, liushuai
comments: true
menu: 关于
permalink: /about/
---

我叫刘帅，我是一个90后的程序员，通过这个博客网站，用它记录学习和生活的点点滴滴，我勉励自己，一定不能偷懒，学无止境。

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
