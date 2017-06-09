---
layout: default
title: Post Archive
meta: The vault of all posts.
---

<div id="blogposts" class="archive">
  <h2>Archive</h2>
  <ul>
    {% for post in site.posts %}
      <li><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a><span>{{ post.date | date_to_string }}</span> </li>
    {% endfor %}
  </ul>

</div>
