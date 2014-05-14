---
layout: page
title: Developer Blog
tagline: Compiling our endeavours
comments: false
---
{% include JB/setup %}

Welcome to the Workday Developer Blog. This blog
does not discuss Workday API's or how you can achieve task 'X in the Workday system, that is already well served by the [Workday Community](http://community.workday.com/). This blog provides a place where we can discuss technologies we are actively researching, using or just excited about. 

<table><tr><td valign="center">Connect with us on Twitter <a href="https://twitter.com/WorkdayDev">@WorkdayDev</a>&nbsp;&nbsp;&nbsp; </td><td><a href="https://twitter.com/WorkdayDev" class="twitter-follow-button" data-show-count="false" data-show-screen-name="false">Follow @WorkdayDev</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script></td></tr></table>

    
## Recent Posts


  {% for post in site.posts %}
<h3><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h3>
<ul class="posts">
           <p>{{ post.excerpt }}</p>
</ul>
  {% endfor %}



