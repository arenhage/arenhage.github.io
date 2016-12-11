---
layout: post
title: Meteor Update Highchart Dynamically
date: 2015-05-19 08:12
author: arenhage
comments: true
categories: [highcharts, Home, meteor]
---
Reactiveness is always key when we are working in Meteor. Hence it would be nice if we could deliver fully reactive charts with little effort. Thankfully this is very easy using the highcharts package!

<!--more-->

<a href="https://atmospherejs.com/maazalik/highcharts" title="Highcharts">Meteor Highcharts</a>

A similar approach will probably be applicable for the other chart libs, but highcharts was very straight fortward, so here goes!

In our template we simply add a container for our alleged chart, in this case im planning on filling it with a pie-chart

```javascript
<div id="piechart" style="width: 200px; height: 200px;"></div>
```

Since we want this chart to be reactive, any changes to our data-source for this chart needs to be take care of. This can easily be achieved bymaking use of the <a href="http://docs.meteor.com/#/full/observe" title="Meteor Observe">Meteor Observe</a>

Basically what observe does is that it observes a query providing a cursor triggering specific events. In our case we want to monitor change events.
Highcharts then provides a very handy method <strong>setData</strong> on its <strong>series</strong> object, which conveniently lets us update the data on for this chart.

```javascript
//make chart
$('#container-pie').highcharts(options);
//monitor changes and update chart
CollectionWithPieInformation.find({}).observe({
  changed: function(newDoc, oldDoc) {
    //newDoc is the full new document including changes.
    var series = $('#container-pie').highcharts().series[0];
    series.setData(newDocument.data);
  }
});
```

Basically this is it, given that our find({}) will return a document with a field <strong>data</strong> matching the format expected by the chart. any state change to the database will be observed by the meteor application and applied to the chart accordingly!

$('#piechart').highcharts(<strong>options</strong>);
how to configure your highchart options can be found in the highchart documentation, or e.g in <a href="http://www.highcharts.com/demo/pie-basic" title="this example" target="_blank">this example</a>
