---
layout: page
title: #
tagline: The Computer Engineer's Guide To The Network
---
{% include JB/setup %}
<a href="https://github.com/you"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://s3.amazonaws.com/github/ribbons/forkme_right_gray_6d6d6d.png" alt="Fork me on GitHub"></a>
<ul class="posts">
{% for post in site.posts limit:5 %}
<article>
<h2>[{{post.category}}] - <em><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></em></h2>
<p>{{ post.content }}</p>
</article>
{% endfor %}</ul>

<h2><a href="archive.html">More Posts</a></h2>

Read [About Me](http://elauqsap.github.io/intro/2013/06/22/about-me/)
