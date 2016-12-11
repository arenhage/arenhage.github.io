---
layout: post
title: Grails Render Pretty JSON
date: 2014-04-09 14:34
author: arenhage
comments: true
categories: [grails, Grails/Groovy, Home, json]
---
Finding this information saved me a lot of time [http://www.intelligrape.com/blog/2012/07/16/rendering-json-with-formatting/](http://www.intelligrape.com/blog/2012/07/16/rendering-json-with-formatting/)

In grails we we often like to render our grails output like so:

```java
render json as JSON
```

<!--more-->

Grails converter will render the json without any formatting, which sometimes (e.glarge json files) can be hard to read.

To render our json in a pretty way we can do the following:
```java
def json = doc as JSON
json.prettyPrint = true
json.render response
```

the <strong>prettyPrint</strong> option returning the grails converter json will also apply for e.g when returning it as an outputstream back to the user.

```java
def json = doc as JSON
json.prettyPrint = true
response.setHeader("Content-disposition","attachment;filename=\"" + fileName + "\"");
response.setContentType("application/json");
response.outputStream << json
```



