---
layout: page
title: Keep learning, keep thinking!
tagline: a place to record my feelings and thoughts
---
{% include JB/setup %}

## Posts (technical and life)

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## About Me

A `programmer` who:

    Once worked in telecommunication world, now work in storage world.
    Focus on Linux platform and now studying kernel.
    C language is always my favorite, Python&Java are also my friends.
    C++ scares me a bit and Perl makes me dizzy :-).


## Contact Me

Mail: [weiweipossible@gmail.com]    QQ: [67931462]    Weibo [@vvpossible]


