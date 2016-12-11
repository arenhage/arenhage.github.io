---
layout: post
title: Grails asynchronous view loading with remoteFunction
date: 2014-02-23 12:15
author: arenhage
comments: true
categories: [grails, Grails/Groovy, Home, javascript, remoteFunction]
---
This is in no way anything new, i just wanted to share an example of a really nice approach (imo) on how you can load some operation heavy rendering asynchronously using the grails remoteFunction.
Assuiming that you have a database with a large dataset that you want to aggregate and render in some fashion on a gsp page , but you do not want to stall the pageload opon navigation.

In our gsp we could add something similiar to this..

<!--more-->

```html
<div>
  <span class="data-loading-spinner"><r:img uri="/images/spinner.gif"/></span>
  <div id="table_data"></div>
</div>
```

Opon pageload we will se a loading spinner waiting to be replaced with the actuall data...
Since we want to load the table data when the page has fully loaded, we do this by issuing a remote call opon document.ready

```javascript
<r:script>
$(document).ready(function() {
    ${remoteFunction(
        controller:'home',
        action:'render_table_data',
	update:'table_data',
	onLoading:'$(".data-loading-spinner").removeClass("hidden");',
	onComplete:'$(".data-loading-spinner").addClass("hidden");'
    )};
});
</r:script>
```

The remoteFunction will show the data-loading-spinner during the issuing call, and add hidden to it as soon as it returns onComplete.

So what happens in the controller?

```java
def render_table_data() {
  //here we do whatever to gather our data.. whether it is calling a service doing some queries to the database...
  //when finished, create a model object and render it using a separate template.
  def model = [table_data:table_data]
  render template:"table_data_template", model:model
}
```

table_data_template here is a separate gsp file that is in the same context as our home view.

```html
<table>
  <thead>
    <tr>
      <th>Name</th>
        <th>Data</th>
    </tr>
  </thead>
  <tbody>
    <g:each in="${table_data}">
      <tr>
        <td>${it.name}</td>
        <td>${it.calculatedData}</td>
      </tr>
    </g:each>
  </tbody>
</table>
```
