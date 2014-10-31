---
layout: default
title: shuaizki 
---

<ul class="posts">
    {% for post in site.posts %}
        {%if post.category == "data_gunner" and post.stared != nil %} <li><span>{{post.date | date_to_string}} </span> &raquo; <a href="{{post.url}}">{{post.title}}</a></li>{%endif%}
    {% endfor %}
    {% for post in site.posts %}
        {%if post.category == "paper_room" and post.stared != nil %} <li><span>{{post.date | date_to_string}} </span> &raquo; <a href="{{post.url}}">{{post.title}}</a></li>{%endif%}
    {% endfor %}
    {% for post in site.posts %}
        {%if post.category == "lucid_dreams" and post.stared != nil %} <li><span>{{post.date | date_to_string}} </span> &raquo; <a href="{{post.url}}">{{post.title}}</a></li>{%endif%}
    {% endfor %}
</ul>
