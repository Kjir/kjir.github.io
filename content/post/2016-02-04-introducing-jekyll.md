---
categories:
- jekyll
date: 2016-02-04T00:14:00Z
tags:
- jekyll
title: Introducing Jekyll!
url: /2016/02/04/introducing-jekyll/
---

Welcome to my new blog, Code Lilies: Beautiful tricks from the swamp.

Like many others before me, I decided to use my personal website to write about
interesting solutions to coding problems that normally arise during software
development. Like Water Lilies that grow through the muddy waters of a swamp, I
will try to share the most beautiful tricks that emerge from the constant flow
of problem our jobs require us to solve every day.

And what could be better than to share how I configured this blog itself? So
let's delve into the details of my Jekyll customizations.

## Jekyll theme

The chosen theme for my blog is Jekyll-Uno by [Josh
Gerdes](https://twitter.com/joshgerdes), a minimalist theme based on Uno for
Ghost, which can be found on
[Github](https://github.com/joshgerdes/jekyll-uno).  I also used some tiny bits
from the [HPSTR](https://mmistakes.github.io/hpstr-jekyll-theme/) theme by
[Michael Rose](https://twitter.com/mmistakes), in particular for inspiration on
how to include comments on my posts.

## Customizations

To better suit my taste, I added a couple of quirks to the Jekyll-Uno theme.

### Link to Facebook profile
Maybe the C00l K1dz only tweet nowadays, but where I live Facebook is the one
your (real) friends are using. So I added the Facebook icon to the social icons
list in the `_includes/header.html` file:

{% raw %}
```html
{% if site.author.facebook_username %}
<!-- Facebook -->
<li class="navigation__item">
  <a href="http://fb.me/{{ site.author.facebook_username }}" title="{{ site.author.facebook_username}} on Facebook" target="_blank">
    <i class="icon icon-social-facebook"></i>
    <span class="label">Facebook</span>
  </a>
</li>
{% endif %}
```
{% endraw %}

and then, of course, I added my Facebook username in the `_config.yml` file:

```yaml
author:
  facebook_username: stephane.bis
```

### Profile image taken from Gravatar
Jekyll-Uno has a generic profile image in `images/profile.jpg`, which you can
substitute with your own. I didn't want to save my image there, though, because
I know that I would forget to update it: I decided instead that I would grab my
profile image directly from Gravatar!

First I added the md5 of my e-mail address in `_config.yml`:

```yaml
author:
  gravatar: b6b7cec33b810ab28ec19be4b4747028
```

Then I modified the header to display the gravatar image, instead of the local
file. So the profile image in `_includes/header.html` looks like this:

{% raw %}
```html
<a href="{{ site.baseurl }}" title="link to home of {{ site.title }}">
  {% if site.author.gravatar %}
  <img src="{{ site.author.gravatar | prepend: '//www.gravatar.com/avatar/' }}?s=100" class="user-image" alt="My Profile Photo">
  {% else %}
  <img src="{{ "images/profile.jpg" | prepend: site.baseurl }}" class="user-image" alt="My Profile Photo">
  {% endif %}
  <h1 class="panel-cover__title panel-title">{{ site.title }}</h1>
</a>
```
{% endraw %}

That was easy!

### Comments with Disqus
Jekyll generates static pages, but that doesn't mean we have to renounce to
comments on our blog! [Disqus](http://www.disqus.com/) allows you to embed an
advanced comment system which works perfectly with Jekyll and Github Pages. To
add the comments I took inspiration (and some code) from the HPSTR Jekyll
Theme, which includes them by default.

First I added a new include in `_includes/disqus.html` which includes the
disqus scripts:

{% raw %}
```html
{% if site.disqus_shortname %}
    <script type="text/javascript">
        /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
        var disqus_shortname = '{{ site.disqus_shortname }}'; // required: replace example with your forum shortname
        var disqus_config = function () {
          this.page.url = '{{ site.url }}{{ page.url }}';
          this.page.identifier = '{{ page.id }}';
          this.page.title = '{{ page.title }}';
        };
        /* * * DON'T EDIT BELOW THIS LINE * * */
        (function() {
            var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
            dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
        })();
        /* * * DON'T EDIT BELOW THIS LINE * * */
        (function () {
            var s = document.createElement('script'); s.async = true;
            s.type = 'text/javascript';
            s.src = '//' + disqus_shortname + '.disqus.com/count.js';
            (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
        }());
    </script>
    <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
    <a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>
{% endif %}
```
{% endraw %}

The only customizations in this file are the the configuration variables for
Disqus which references some setting in our `_config.yml` file. I also added a
new configuration option:

```yaml
disqus_shortname: 'my-disqus-shortname'
```

And at last I add the comments section on the post layout file in
`_layouts/post.html`, right before the closing of the `<article>` tag:

{% raw %}
```html
  {% if page.comments != false and site.disqus_shortname %}
  <section id="disqus_thread"></section><!-- /#disqus_thread -->
  {% endif %}
</article>
{% if page.comments != false %}
{% include disqus.html %}
{% endif %}
```
{% endraw %}

Don't believe me? Leave me a comment at the end of the article! :)

### Default layout for posts

In Jekyll-uno you have to set the layout for posts in the YAML Front Matter. I
didn't like that, so I decided to add a different default for posts, so that I
may be free to forget about setting it on new posts. Here is what I added at
the end of my `_config.yml`:

```yaml
defaults:
  -
    scope:
      path: ""
      type: "posts"
    values:
      layout: "post"
```

## Conclusion

These are the first simple customizations I implemented for my blog. Of course,
I also contributed back all of my changes to the original theme, and you can
find all the code I wrote about here in my [pull
requests](https://github.com/joshgerdes/jekyll-uno/pulls?utf8=%E2%9C%93&q=is%3Apr+author%3AKjir+).

What do you think about these changes? Were they useful? Could they be
improved? Please leave a comment with your feedback!
