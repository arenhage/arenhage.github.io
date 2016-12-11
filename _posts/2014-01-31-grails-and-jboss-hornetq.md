---
layout: post
title: Grails and JBoss HornetQ
date: 2014-01-31 18:49
author: arenhage
comments: true
categories: [grails, Grails/Groovy, Home, HornetQ, JBoss, JMS]
---
I would just like to share a setup i did a while back connecting my Grails application(app.grails.version=2.2.3) sending JMS messages to a JBoss HornetQ.

There were some information but not a lot on different approaches in setting up the connection factories, some more extensive than others...

Any way... The way i ended up doing it was to define my <strong>HornetQJMSConnectionFactory</strong> as a bean defined in resources.xml. I can then use this bean to instantiate a new <strong>JmsTemplate</strong> producer which then an be used to actually send the messages.

<!--more-->

<span style="text-decoration:underline;">resources.xml</span>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="hornetConnectionFactory" class="org.hornetq.jms.client.HornetQJMSConnectionFactory">
        <constructor-arg name="ha" value="false" />
        <constructor-arg>
            <bean class="org.hornetq.api.core.TransportConfiguration">
                <constructor-arg value="org.hornetq.core.remoting.impl.netty.NettyConnectorFactory" />
                <constructor-arg>
                    <map key-type="java.lang.String" value-type="java.lang.Object">
                        <!-- HornetQ standalone instance details -->
                        <entry key="host" value="localhost"></entry>
                        <entry key="port" value="5445"></entry>
                    </map>
                </constructor-arg>
            </bean>
        </constructor-arg>
    </bean>

    <!-- ConnectionFactory Definition -->
    <bean id="connectionFactory" class="org.springframework.jms.connection.CachingConnectionFactory">
        <constructor-arg ref="hornetConnectionFactory" />
    </bean>

    <bean id="hornetqJmsConfig" class="org.apache.camel.component.jms.JmsConfiguration">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="transacted" value="false" />
        <property name="requestTimeout" value="60000" />
    </bean>

    <bean id="hornetq" class="org.apache.camel.component.jms.JmsComponent">
        <property name="configuration" ref="hornetqJmsConfig" />
    </bean>
</beans>

```

Now when we have our spring bean in our context we can then from a e.g controller och service do

```java
class SendMessageService {
  def hornetConnectionFactory

  def sendMessage() {
    JmsTemplate producer = new JmsTemplate(hornetConnectionFactory);
    producer.send(new HornetQQueue("myqueue"), new MessageCreator() {
      @Override
      public Message createMessage(Session session) throws JMSException {
        // Create a byte messages
        BytesMessage message = session.createBytesMessage()
        message.writeBytes("hello world".getBytes())
        return message
      }
    }
  }
}
```
    
