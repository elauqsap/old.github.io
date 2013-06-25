---
layout: page
---
{% include JB/setup %}
<a href="https://github.com/pasqualedagostino"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://s3.amazonaws.com/github/ribbons/forkme_right_red_aa0000.png" alt="Fork Me on GitHub"></a>
### The Computer Engineer's Guide To The Network
<ul class="posts">
{% for post in site.posts limit:5 %}
<article>
<h2>[{{post.category}}] - <em><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></em></h2>
<p>{{ post.content }}</p>
</article>
{% endfor %}content</ul>

<h2><a href="archive.html">More Posts</a></h2>

Test [Project](http://pasqualedagostino.github.io/tds/index.html)
Read [About Me](http://pasqualedagostino.github.io/intro/2013/06/22/about-me/)
