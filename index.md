---
layout: home
theme: jekyll-theme-minimal
---

# Добро пожаловать в мой блог!
Ниже представлены мои заметки:

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
