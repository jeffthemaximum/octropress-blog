---
layout: post
title: "Flask tutorial: SQLite3 databse"
date: 2015-07-24 14:14:35 -0400
comments: true
categories: 
published: false
---

Friday's are "job-prep" days at [Recurse Center](www.recurse.com). We get together to practice the types of projects or challenge questions that we might get at a technical programming interview. This past Friday, the task was this: 
<!-- more -->
"Build a website like [Bitly](www.bitly.com) that allows you to submit a link, then gives you a new, hopefully shorter url to use for that link."

I'm going to walk you through how to solve this challenge using Python, the [Flask microframework](http://flask.pocoo.org/), and an SQLite3 database, and we'll try to host the whole thing on [heroku](www.heroku.com).

To get started, you want to launch a basic flask app using [this tutorial](http://code.tutsplus.com/tutorials/an-introduction-to-pythons-flask-framework--net-28822).

Following that tutorial, you should have a project structure that looks like this:

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>&#9500;&#9472;&#9472; app
&#9474;   &#9500;&#9472;&#9472; static
&#9474;   &#9474;   &#9500;&#9472;&#9472; css
&#9474;   &#9474;   &#9500;&#9472;&#9472; img
&#9474;   &#9474;   &#9492;&#9472;&#9472; js
&#9474;   &#9500;&#9472;&#9472; templates
&#9474;   &#9500;&#9472;&#9472; routes.py
&#9474;   &#9492;&#9472;&#9472; README.md
</code></pre>

In future posts, I'll outline how I progressed from the structure outlined above to a functional URL shortener. Stay tuned for that! I will walk you through how I went from the structure above to a functional URL shortening flask app that has the following structure:

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>&#9500;&#9472;&#9472; app
&#9474;   &#9500;&#9472;&#9472; static
&#9474;   &#9474;   &#9500;&#9472;&#9472; css
&#9474;   &#9474;   &#9474;   &#9500;&#9472;&#9472; main.css
&#9474;   &#9474;   &#9500;&#9472;&#9472; img
&#9474;   &#9474;   &#9492;&#9472;&#9472; js
&#9474;   &#9500;&#9472;&#9472; templates
&#9474;   &#9474;   &#9500;&#9472;&#9472; home.html
&#9474;   &#9474;   &#9500;&#9472;&#9472; layout.html
&#9474;   &#9474;   &#9492;&#9472;&#9472; tinyURL.html
&#9474;   &#9500;&#9472;&#9472; routes.py
&#9474;   &#9492;&#9472;&#9472; README.md
</code></pre>

For the purposes of this post, I'll just be focusing on how I added SQLite3 to my URL shortener flask app of the structure above.

At this point, users visit my my URL shortening website, enter a URL, and my flask app stores that URL in a python dictionary