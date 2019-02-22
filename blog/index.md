---
layout: single-page
---

<ul class="post-list">
    {% for post in site.posts limit 4 %}
    <li><a class="post-title" href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
    <br><span class="post-meta-preview">{{ post.date | date_to_string }} • Filed under: {% for category in post.categories %}
          <a href="{{site.baseurl}}/categories/#{{category|slugize}}">{{category}}</a>
            {% unless forloop.last %},{% endunless %}
              {% endfor %}</span></li>
        {{ post.content | strip_html | truncatewords:75}}<br>
            <a href="{{ post.url }}">Read more...</a><br>
    <hr>
    {% endfor %}
</ul>
