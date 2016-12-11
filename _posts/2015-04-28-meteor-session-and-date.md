---
layout: post
title: Meteor session and Date
date: 2015-04-28 15:17
author: arenhage
comments: true
categories: [date, Home, meteor, session]
---
Not 100% sure this has been included or not in the most recent release of Meteor, but for a time there was a problem when trying to store dates in the Meteor Session object due to JSON-compatibility issues and the hot code push.
For those whos still running on a version experiencing problems storing dates in session, the way i worked around this was simply to use moment with two helper methods.

```javascript
getChosenDate = function() {
  return moment(Session.get('chosenDate'));
};

setChosenDate = function(date) {
  Session.set('chosenDate', date.valueOf());
};
```
