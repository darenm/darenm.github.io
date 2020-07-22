---
layout: post
title:  "How to Embed YouTube Videos in Markdown"
date:   2020-06-23 12:00:00 +0700
categories: custommayd "developer training" videos markdown
---
I can never remember the best way to embed a video in Markdown, so here, for posterity is the information.

If we combine the markup for an embedded image and a link, we can  insert an image with a link:

```
[![alt text](http://darenmay.com/headshot.png)](http://darenmay.com/post "post title")
```

Taking this approach, we can create clickable images that navigate to a YouTube video:

```
[![Blinking LEDs](http://img.youtube.com/vi/XAMVzS13HY0/0.jpg)](http://www.youtube.com/watch?v=XAMVzS13HY0 "Blinking LEDs")
```

Which results in:

[![Blinking LEDs](http://img.youtube.com/vi/XAMVzS13HY0/0.jpg)](http://www.youtube.com/watch?v=XAMVzS13HY0 "Blinking LEDs")


There is a great app that does this for you: <http://embedyoutube.org/>