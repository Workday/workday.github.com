---
layout: page
title: Developer Blog
tagline: Compiling our endeavours
comments: false
---
{% include JB/setup %}

Welcome to the Workday Developer Blog. This blog
does not discuss Workday API's or how you can achieve task 'X in the Workday system, that is already well served by the [Workday Community](http://community.workday.com/). This blog provides a place where we can discuss technologies we are actively researching, using or just excited about. 

You can connect with us on Twitter [@WorkdayDev](https://twitter.com/WorkdayDev) 
    
## Recent Posts


  {% for post in site.posts %}
<h3><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h3>
<ul class="posts">
           <p>{{ post.excerpt }}</p>
</ul>
  {% endfor %}



