---
layout: post
title: Send SVG to server
date: 2014-06-02 11:29
author: arenhage
comments: true
categories: [Home, javascript, SVG]
---
Using javascript, I ended up just parsing my svg html as a string and sending it with my post data.

```javascript
var svg = document.getElementsByTagName('svg')[0];
var xml = (new XMLSerializer()).serializeToString(svg);
```

Include it in your post data and send it to the server...
