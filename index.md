---
layout: home
---

# Добро пожаловать в мой блог!
Ниже представлены мои заметки:

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> 
      — {{ post.date | date_to_string }}
    </li>
  {% endfor %}
</ul>
