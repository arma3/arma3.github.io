---
title: Home
layout: default
order: 0
sitemap:
    exclude: "yes"
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

```js
 
private _a = "test"; 
 
``` 