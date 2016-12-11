---
layout: post
title: Abstract Security Filter on Controllers
date: 2014-05-23 11:03
author: arenhage
comments: true
categories: [filters, grails, Grails/Groovy, Home]
---
What i wanted to achieve was a generic way of determining if a user in my system was suppose to have access to a certain part of the system. I'm sure there are some really nice plugins or stuff that achieves this, but the way i achieved this functionality was to implement a controller based access filter. This way i can limit all actions in that controller simply by defining <strong>action: '*'</strong>

<!--more-->

```java
def filters = {
  all(controller: '*', action: '*') {
    before = {
      def privileged = ......//in what ever way we get ahold of this value (e.g database or session)
      def controllerArtefact = grailsApplication.getArtefactByLogicalPropertyName("Controller", "$controllerName")
      def controllerClass = controllerArtefact.getClazz()
      try {
        if(controllerClass?.ENTITY_CLASS.equals("LIMITED-ACCESS")) {
          if(!privileged) {
            redirect(controller: "unauthorized", action:"unauthorized")
          }
        }
      }
      catch(MissingPropertyException e) {
        //something appropriate
      }
    }
  }
}
```

The only thing left to do is to add the property ENTITY_CLASS (or what ever you want to name it) to your controller which you want to limit access to.

static final ENTITY_CLASS = 'LIMITED-ACCESS'
