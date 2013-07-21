---
layout: post
tags: [ blog, jekyll, github:pages ]
title: Hello World!
---

The time has come for me to venture into the blogging business. Every day I deal with various software related conecpts, 
ideas, and problems. I learn from these interactions a lot. What bothers me is that now, after 6 years of professional 
software development I realize that I don't really remmber what I've been working on in the past years. There are lots of 
grey areas even if I sit down and revisit some of my past code. In other words the retention isn't that great.

In this blog I'm going to regularly write on the aforementioned topics. And I hope that I will get better idea about 
them after formalizing my findings in writing.

For this purpose I set up this blog. It was a challenge of its own to make a platform choice. Nowardays there is a 
ton of different hosted blogging platforms - Wordpress, Blogger, Tubmblr to name a few. But I happened to stumble upon 
[GiHub:Pages](http://pages.github.com/) and [Jekyll](https://github.com/mojombo/jekyll). This neat combo currently 
drives my blog.

Jekyll is a blow aware static site generator. It runs on a bunch of templated files and outputs static HTML which can 
be hosted virtually anywhere, and on GitHub:Pages in particular. GitHub:Pages will run Jekyll begind the scenes, 
and serve the generated site. My job was to set up all the templates and configuration. There is a few prebuilt solutions 
available for this purpose - [Jekyll-Bootstrap](http://jekyllbootstrap.com/) or 
[Octopress](https://github.com/imathis/octopress/), but I wanted to go throught this process myself, and build it from
scratch. Of course my implementation doesn't have too many features, but it's mine :)

In a few words - Jekyll grabs HTML, Markdown, or Textile files, passes them through the specified template and outputs a
plain HTML file. The beauty of this approach is that I can write posts in my favorite editor - Vim and just push them to
my GitHub repo. GitHub:Pages will then call Jekyll on the collection of files, regenerate the site and promptly publish it.

Such a blog is blazing fast, the hosting is free, and the amount of control over the content and markup is just
incredible. And [Twitter Bootstrap](http://twitter.github.com/bootstrap/) has everything a wannabe gloggers needs to create
pretty posts.

The source code for this blog [rests here](https://github.com/akhodakivskiy/akhodakivskiy.github.com)

Hope that wasn't too bad for the first time :)
