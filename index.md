---
layout: page
---
{% include JB/setup %}
<h2>The Computer Engineer's Guide To The Network</h2>
<ul class="posts">
{% for post in site.posts limit:5 %}
<article>
<h2>[{{post.category}}] - <em><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></em></h2>
<p>{{ post.content }}</p>
</article>
{% endfor %}			
</ul>

<h2><a href="archive.html">More Posts</a></h2>

Read [About Me](http://pasqualedagostino.github.io/intro/2013/06/22/about-me/)
