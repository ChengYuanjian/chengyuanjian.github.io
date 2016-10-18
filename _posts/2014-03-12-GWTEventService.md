---
layout: post
title: GWTEventService介绍
description: GWTEventService is an event-based client-server communication framework. It uses GWT-RPC and the Comet / server-push technique. 
keywords: gwt, google
category: GWT
tags: [GWT,Google]
---

GWTEventService is an event-based client-server communication framework. It uses GWT-RPC and the Comet / server-push technique. The client side offers a high-level API with opportunities to register listeners to the server like to a GUI component. Events can be added to a context/domain on the server side and the listeners on the client side get informed about the incoming events. The server side is completely independent of the client implementation and is highly configurable. Domains can be defined to decide which events are important for the different contexts. 

<!-- more -->

###Advantages

* Encapsulation of the client-server communication
* High-level API with listeners and events
* Only one open connection for event listening
* Reduction of server calls
* Reduction of connection peaks
* Events are returned directly when the event has occurred (instead of polling)
* Events are bundled to reduce server calls
* Server-side event filtering to reduce server calls
* Based on the GWT-RPC mechanism
* Automatic timeout recognition and handling
* Extensible architecture 

###Resouce

https://code.google.com/p/gwteventservice/
