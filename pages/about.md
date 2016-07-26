---
layout: page
title: About
description: >
  人的一生要疯狂一次，无论是为一个人，一段情，一段旅途，或一个梦想。
  我们要敢于背上超出自己预料的包袱，努力之后，你会发现自己要比想象的优秀很多。
  放下你的三分钟热度，放空你禁不住诱惑的大脑，放开你容易被任何事物吸引的眼睛，放淡你什么都想聊两句八卦的嘴巴，静下心来好好做你该做的事，该好好努力了！
  记住一句话：越努力，越幸运。
keywords:  蜗牛，关于我，我的介绍
comments: true
menu: 关于
permalink: /about/
---

## 坚信

* 熟能生巧
* 努力改变人生

## 联系

* GitHub：[@johnstevin](https://github.com/johnstevin)
* 博客：[{{ site.title }}]({{ site.url }})
* 微博: [@AbangSir](http://weibo.com/AbangSir)
* 知乎: [@cool2rong](https://www.zhihu.com/people/cool2rong)
* 邮箱：[@Email](mailto://stevin.john@qq.com)

## Skill Keywords

#### Software Engineer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_software_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Mobile Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_mobile_app_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Windows Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_windows_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>
