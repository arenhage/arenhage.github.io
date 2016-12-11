---
layout: post
title: Presentation-poll
date: 2016-02-28 14:09
author: arenhage
comments: true
categories: [Home, meteor]
---

Some weeks back i was given an idea from a colleague of mine, they were going to have a workshop with a larger group of people and needed some way to keep track of people were on the right track following along.
This sprung the idea for the little application i now have created namely [http://presentation-poll.com](http://presentation-poll.com)

<!--more-->

Its a very tiny application, but it gets the job done! And also i got to learn some new stuff regarding the DDP Server/Client handling in Meteor which opened up some new ideas for this application.

What it was initially intended for was to keep track of a unique vote from a single individual and present the result in real-time.

Initially, in order for me to keep track of every individual vote (yes/no/blank), each participant had to state some form of name on which i could store unique information including his/her current vote. And with Meteors real-time architecture, this could be reflected on the common result accordingly. And this worked fine, but i wanted to simplify it even further.

So i turned to the idea of only keeping track of this based on active connections to the current poll, but with the exception that i did not want to introduce any other third party web-socket libraries in order to achieve this. Since Meteor is working with websockets continuously i was hoping that i could achieve this using what is already given to me by Meteor core.

And it turns out that i was able to achieve this by hooking onto the DDP server as well as checking the current connection of the client, fairly simple.

Basically what i do is that whenever a client jumps into a specific poll. The client will register its last known session id by looking at its current websocket connection id to the DDP server. Like so:

```javascript
Meteor.connection._lastSessionId
```

The only other thing i needed to keep track of was when ever a client dropped its connection towards the DDP server, deregister it from the current poll and drop its vote if a vote was cast so that the calculations is done on the correct number of connected clients.

This was also easily achieved by simply stating that for any connection made to the DDP server, register an onClose for the connection. When onClose is called (when the connection is dropped) simply deregister it from the poll, removing its vote and data. For example:

```javascript
Meteor.onConnection(function(connection)  {
  connection.onClose(function() {
    SomeCollection.remove({sessionId:connection.id});
  });
});
```

