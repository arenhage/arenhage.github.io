---
layout: post
title: Grails truncate taglib
date: 2014-04-02 13:40
author: arenhage
comments: true
categories: [ellipsis, grails, Grails/Groovy, gsp, Home, truncate]
---
Ran into this taglib created by <a href="https://gist.github.com/acreeger" title="Adam Creeger">Adam Creeger</a> which lets you truncate strings in your grails views.
It was exactly what i was looking for except for the missing possibility of also clicking it to extend and show the full text. Here is my updated version for achieving this.

<!--more-->

```java
class TruncateTagLib {
private static final String ELLIPSIS = '...'
  def truncate = { attrs, body ->
    def maxLength = attrs.maxlength
    if (maxLength == null || !maxLength.isInteger() || maxLength.toInteger() <= 0) {
      throw new Exception("The attribute 'maxlength' must an integer greater than 3. Provided value: $maxLength")
    } else {
      maxLength = maxLength.toInteger()
    }
    if (maxLength <= ELLIPSIS.size()) {
      throw new Exception("The attribute 'maxlength' must be greater than 3. Provided value: $maxLength")
    }
    if (body().length() > maxLength) {
        out << /<abbr id="$attrs.id" title="${body().encodeAsHTML()}" class="ellipsis"
                onclick="javascript:if($('#$attrs.id').hasClass('ellipsis')) {
                    $('#$attrs.id').html('${body()}'); $('#$attrs.id').removeClass('ellipsis')
                } else {
                    $('#$attrs.id').html('${body()[0..maxLength - (ELLIPSIS.size() + 1)].encodeAsHTML()}$ELLIPSIS');
                    $('#$attrs.id').addClass('ellipsis') }">
            ${body()[0..maxLength - (ELLIPSIS.size() + 1)].encodeAsHTML()}$ELLIPSIS<\/abbr>/
    } else {
      out << body().encodeAsHTML()
    }
  }
}
```

and in your view gsp simply use it as:

```java
<g:truncate id="abbr_${it?.id}" maxlength="50">${it?.text}</g:truncate>
```

where <strong>it?.id</strong> is a unique identifier
