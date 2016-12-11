---
layout: post
title: Meteor, rapid and fun!
date: 2015-03-10 10:20
author: arenhage
comments: true
categories: [Home, meteor]
---
So the last couple of days i have been exploring the world of <a title="Meteor" href="https://www.meteor.com/" target="_blank">Meteor</a>, a platform for building mobile and web applications in pure javascript. The undeniable conclusion is, its rapid, rapid fast development for those already comfortable with javascript.

<!--more-->

But more interesting is the way we think when we are programming meteor. The reactive thinking using the publish/subscribe model is very powerful if you do it right, but that doesnt necessarily mean that you do. If you are not familiar with publish/subscribe which i would like to say originate from the world of messaging, it can be some what of a threshold to surpass, but once you wrap your head around it, it will bring you a lot of interesting concepts to explore.

The only thing so far that i have found quite devious is the native problem with javascript, the load order of the files. Meteor tries to solve this by introducing a "special folders" concept, where folders containing your script files are intended to be loaded before another. However i can only imagine what will happen when your project grows larger.

In any case! Meteor is great fun, and i think this is only the tip of the iceberg with this style of programming. Cant wait to see where this is headed!

For those interested, my reference project was to create a traffic service which collects data from a public api, displaying it in a graph and make it mobile friendly. Thi intention is to notice the peak times when the destinations are crowded, so that you can plan ahead accordingly! Check it out if you want to see it in action!

<a title="http://trafikval.com/" href="http://trafikval.com/" target="_blank">http://trafikval.com/</a>
