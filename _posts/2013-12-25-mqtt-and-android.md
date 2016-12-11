---
layout: post
title: MQTT and Android
date: 2013-12-25 17:32
author: arenhage
comments: true
categories: [Android, Home, Java, MQTT, OrmLite]
---
A while back a friend and started developing an online multiplayer Geo-Racing game where a group of people are able to compete against each other in a race from Point A to Point B anywhere in the world (almost). In all its simplicity it is basically like any other running gps tracker out there, with an additional built in competitive system. This post is simply just a small introduction and overview of some of the technologies that is included in the project and a brief introduction of the backend mechanics of the game.

<!--more-->

My project partner has written a nice post with some more detail on the technology layers and some more details on the beneficial common layering :

<a href="http://www.squeed.com/blog/2013/12/mqtt-powered-event-bus-for-android-applications/">http://www.squeed.com/blog/2013/12/mqtt-powered-event-bus-for-android-applications/</a>

<a href="http://www.squeed.com/blog/2013/12/common-jpa-entities-for-android-and-backend/">http://www.squeed.com/blog/2013/12/common-jpa-entities-for-android-and-backend/</a>

<strong>Why MQTT?</strong>

Because we thought it was a fun idea exploring the possibilities of creating a fully message driven system on Android. Also with its messaging being tiny, power efficient, low cost keep alive and low latency push capabilities, it is a perfect fit for the system we are building.

I will not touch upon the subject of performance, there is already an excellent post from Stephen Nicholas taking up some of these aspects <a href="http://stephendnicholas.com/archives/219">http://stephendnicholas.com/archives/219</a>.

<strong>Technology choices</strong>

There are more and more brokers out there supporting the MQTT protocol. Some of the brokers we initially tried out was <strong>ActiveMQ</strong>, <strong>ActiveMQ Apollo </strong>and<strong> Mosquitto. </strong>The natural choice fell on Mosquitto given it is a native MQTT broker with good documentation, a hot mailing list and fast turnarounds for bugs and support (<a href="http://mosquitto.org/">http://mosquitto.org/</a>).

I'v come across two settled opensource MQTT client libraries that can be used for the server and client setup.
<ul>
	<li>Eclipse Paho (<a href="http://www.eclipse.org/paho/">http://www.eclipse.org/paho/</a>) "is the primary home of the reference MQTT clients that started at IBM", has good documentation and supports a variety of languages</li>
	<li>Fusesource MQTT - Client (<a href="https://github.com/fusesource/mqtt-client">https://github.com/fusesource/mqtt-client</a>) (the one used in our project) in all its simplicity has some nice predefined ways of setting up your connecting, such as, <span style="font-family:Consolas, Monaco, monospace;font-size:12px;line-height:18px;">BlockingConnection - FutureConnection and CallbackConnection</span></li>
</ul>
<strong style="line-height:1.5em;">The topology</strong>

<a href="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/untitled-drawing2.png"><img class="alignnone size-medium wp-image-81" alt="Architecture" src="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/untitled-drawing2.png?w=300" width="300" height="195" /></a>

So the architecture of this whole setup is pretty straight forward. Basically what we have from start-up is a <em>Server</em> application connecting and subscribing to an (what we call it) <em>Action Topic (AT) </em>on the Mosquitto broker. Any android application (A) connecting to the broker will start subscribing to a unique topic for that user. If we have 100 users, we will have 101 topics in the system, that may or may not actively subscribe to their own topic. This means that any action that results in a message delivery to one to many topics will result in a very fast push of information to all the consuming devices.

<strong>The message handling</strong>

Has been abstracted as a common library included by both the server and the android applications. Since the MQTT message payload is of type byte, we made the decision to define any message sent between the client and the server (and vice versa) to be consisting of a byte[] serialized java class. This consists of a top level abstract class "<em>Application Message</em>" which any new message type would then extend. This means that any payload sent between the client and server will be a deserializable byte payload known to both the client and server message handler.

So sending a message from the client to the server could go like this:
<ol>
	<li>Create a new message and add all necessary information</li>
	<li>Serialize the message to a byte[]</li>
	<li>Add the data to the buffer payload and publish mqtt message on destination topic.</li>
</ol>
Now the server will retrieve this buffer payload, deserialize it, and by <em>Visitor Pattern </em>determine what kind of "<em>Application Message</em>" where sent and operate on the message accordingly.
The beauty of this now is that the Android application will react in the exact same way. The three steps would be taken by the server for any message that might be sent to the client, and by convention the client subscribing to that topic would determine what message were sent by deserializing and dropping it through the message visitor and act on it accordingly. And for any additional processing necessary on either the server or android application we can simply override the basic behavior for the visit method for that message type and define our own logic.

<strong>The database back-end </strong>has also been included in the common packaging for both the server and the client. OrmLight (<a href="http://ormlite.com/">http://ormlite.com/</a>) is a relatively lightweight ORM java package for persisting java objects in an SQL database. Here we define all our database entities by JPA annotations and define our DAO layer. The beauty of this is that we are simply able to make use of the same dao for both the server and android back-end. Since the servers main objective is "only" to mirror and persist the state of our complete network the way we define our CRUD can be commonly shared.

<strong>The servers </strong>main objective is to persist and store information. In some scenarios it also is responsible for forwarding messages between users when they lack information to send the message directly to the user themselves. It is also responsible for <em>Registration</em> and <em>Login. </em>Since MQTT by nature is asynchronous, we have implemented a custom synchronous messaging between the client and server using callbacks for handling logins and registration.

<strong>The android client </strong>will periodically push its information to one or more topics when active. The messages sent by the android app will be published to either client, client&server or server. When publishing the information, MQTT supports three different types of Quality Of Service (QoS). When choosing in what QoS the message should be sent, there will be an affect on the applications battery consumption etc since there are more or less messages being sent in the background.

Information below on QoS is taken from the paho documentation: <a href="http://www.eclipse.org/paho/files/mqttdoc/Cclient/qos.html">http://www.eclipse.org/paho/files/mqttdoc/Cclient/qos.html</a>
<ul>
	<li><b>QoS0, At most once:</b> The message is delivered at most once, or it may not be delivered at all. Its delivery across the network is not acknowledged. The message is not stored. The message could be lost if the client is disconnected, or if the server fails. QoS0 is the fastest mode of transfer. It is sometimes called "fire and forget". The MQTT protocol does not require servers to forward publications at QoS0 to a client. If the client is disconnected at the time the server receives the publication, the publication might be discarded, depending on the server implementation.</li>
	<li><b>QoS1, At least once:</b> The message is always delivered at least once. It might be delivered multiple times if there is a failure before an acknowledgment is received by the sender. The message must be stored locally at the sender, until the sender receives confirmation that the message has been published by the receiver. The message is stored in case the message must be sent again.</li>
	<li><b>QoS2, Exactly once:</b> The message is always delivered exactly once. The message must be stored locally at the sender, until the sender receives confirmation that the message has been published by the receiver. The message is stored in case the message must be sent again. QoS2 is the safest, but slowest mode of transfer. A more sophisticated handshaking and acknowledgement sequence is used than for QoS1 to ensure no duplication of messages occurs.</li>
</ul>
<strong>Message aggregation</strong>

In all its simplicity we want to assure that any message sent will reach its destination. How we propagate the messages in order for all involved parties to receive their messages is a challenge. The following is an example of how the message propagation works when we do a race-invite to multiple users.

<a href="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/race-invite1.png"><img class="size-medium wp-image-88" alt="race invite1" src="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/race-invite1.png?w=300" width="300" height="228" /></a>

A1 sends a race invite to A2 and A3 containing the race information and a list of anyone that might need further information regarding this race (information so that A2 and A3 may communicate since they might not be friends)

<a href="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/race-invite2.png"><img class="size-medium wp-image-89 " alt="race invite2" src="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/race-invite2.png?w=300" width="300" height="232" /></a>

A2 Accepts the invitation by creating the race in its local database, return his information to A1, A3 and the server so that the two get all information and the server can persist.

<a href="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/race-invite3.png"><img class="size-medium wp-image-90  " alt="race invite3" src="{{ site.baseurl }}/images/2013-12-25-mqtt-and-android/race-invite3.png?w=300" width="300" height="231" /></a>

Finally A3 also accepts the race invite, creating the race in his local database and sending his information to both A1, A2 and the server for persistence.

Whats left out in these examples is the actual retry policy that is underlying for both the server and the android client. There is no support in MQTT (AFAIK) that takes care of persisting and automatically retrying to publish any failed publications. Lets say that the mosquitto broker were to be offline at the point of a message publication, the retry to send this message needs to be handled on client level, hence we have written our own retry policy to take care of these events.

<strong>To sum up...</strong>

So far having chosen MQTT as our primary communications protocol it has been  a fast and fun experience learning some new aspects diverging from the more common ways of communicating in a distributed environment. The project is still in working progress, so we are still eager for any interesting problems that might arise.
