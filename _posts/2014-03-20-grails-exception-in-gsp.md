---
layout: post
title: Grails exception in gsp
date: 2014-03-20 08:38
author: arenhage
comments: true
categories: [exception, grails, Grails/Groovy, gsp, Home]
---
I just came across a neat way of handling potential exceptions for when using taglibs or similar in your gsp code.

In my problem scenario i had defined some generic (twitter-bootstrap) modals that were to be pre-rendered with some form data based on a template located in each controller view.
The only problem was that this template was not used in each and every controller view, resulting in that navigating in some controllers raised an exception due to missing templates.

<!--more-->

The way i approached this was to wrap the portion that does the rendering of the template form in a try-catch block, hence catching any exceptions raised due to missing the form.

{% raw %}
```html
<div id="add" class="modal hide fade">
  <div class="modal-header">
    <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
    <h3>Add</h3>
  </div>
  <div class="modal-body">
    <%
      try { %>
        ${render(template:"form") }
      <%} catch(Exception e){%>
      <%}
     %>
  </div>
  <div class="modal-footer">
    <button data-dismiss="modal" aria-hidden="true">Close</button>
    <button id="btn-save">Save changes</button>
  </div>
</div>
```
{% endraw %}
