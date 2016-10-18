---
layout: post
title: Getting started with BlazeDS
description: This article introduces the BlazeDS framework.
keywords: Flex, BlazeDS
category: Flex
tags: [Flex,BlazeDS]
---

##Introduction to BlazeDS

BlazeDS provides a set of services that lets you connect a client-side application to server-side data, and pass data among multiple clients connected to the server.BlazeDS implements real-time messaging between clients.

A BlazeDS application consists of two parts: a client-side application and a server-side J2EE web application. The following figure shows this architecture:

![blazeds_client_server](http://livedocs.adobe.com/blazeds/1/blazeds_devguide/images/blazeds_client_server.png)

<!-- more -->

###The client-side application

A BlazeDS client application is typically an Adobe Flex or AIR application. Flex and AIR applications use Flex components to communicate with the BlazeDS server, including the RemoteObject, HTTPService, WebService, Producer, and Consumer components. The HTTPService, WebService, Producer, and Consumer components are part of the Flex Software Development Kit (SDK).

Although you typically use Flex or AIR to develop the client-side application, you can develop the client as a combination of Flex, HTML, and JavaScript. Or, you can develop it in HTML and JavaScript by using the Ajax client library to communicate with BlazeDS.

###The BlazeDS server

The BlazeDS server runs in a web application on a J2EE application server. BlazeDS includes three preconfigured web applications that you can use as the basis of your application development. 

Configure an existing J2EE web application to support BlazeDS by performing the following steps:

* Add the BlazeDS JAR files and dependent JAR files to the WEB-INF/lib directory.
* Edit the BlazeDS configuration files in the WEB-INF/flex directory.
* Define MessageBrokerServlet and a session listener in WEB-INF/web.xml.

###BlazeDS features 

The following figure shows the main features of BlazeDS:
![blazeds_features](http://livedocs.adobe.com/blazeds/1/blazeds_devguide/images/blazeds_features.png)

###BlazeDS core features

The BlazeDS core features include the RPC services and the Messaging Service.

####RPC services

The Remote Procedure Call (RPC) services are designed for applications in which a call and response model is a good choice for accessing external data. RPC services let a client application make asynchronous requests to remote services that process the requests and then return data directly to the client. You can access data through client-side RPC components that include HTTP GET or POST (HTTP services), SOAP (web services), or Java objects (remote object services).

Use RPC components when you want to provide enterprise functionality, such as proxying of service traffic from different domains, client authentication, whitelists of permitted RPC service URLs, server-side logging, localization support, and centralized management of RPC services. BlazeDS lets you use RemoteObject components to access remote Java objects without configuring them as SOAP-compliant web services.

A client-side RPC component calls a remote service. The component then stores the response data from the service in an ActionScript object from which you can easily obtain the data. The client-side RPC components are the HTTPService, WebService, and RemoteObject components.

Note: You can use Flex SDK without the BlazeDS proxy service to call HTTP services or web services directly. You cannot use RemoteObject components without BlazeDS or ColdFusion.

####Messaging Service

The Messaging Service lets client applications communicate asynchronously by passing messages back and forth through the server. A message defines properties such as a unique identifier, BlazeDS headers, any custom headers, and a message body.

Client applications that send messages are called message producers. You define a producer in a Flex application by using the Producer component. Client applications that receive messages are called message consumers. You define a consumer in a Flex application by using the Consumer component. A Consumer component subscribes to a server-side destination and receives messages that a Producer component sends to that destination.

The Messaging Service also supports bridging to JMS topics and queues on an embedded or external JMS server by using the JMSAdapter. Bridging lets Flex client applications exchange messages with Java client applications. 

###Service adapters

BlazeDS lets you access many different persistent data stores and databases including JMS, and other data persistence mechanisms. A service adapter is responsible for updating the persistent data store on the server in a manner appropriate to the specific data store type. The adapter architecture is customizable to let you integrate with any type of messaging or back-end persistence system.

###The message-based framework

BlazeDS uses a message-based framework to send data back and forth between the client and server. BlazeDS uses two primary exchange patterns between server and client. In the first pattern, the request-response pattern, the client sends a request to the server to be processed. The server returns a response to the client containing the processing outcome. The RPC services use this pattern.

The second pattern is the publish-subscribe pattern where the server routes published messages to the set of clients that have subscribed to receive them. The Messaging Service uses this pattern to push data to interested clients. The Messaging Service also uses the request-response pattern to issue commands, publish messages, and interact with data on the server.

####Channels and endpoints

To send messages across the network, the client uses channels. A channel encapsulates message formats, network protocols, and network behaviors to decouple them from services, destinations, and application code. A channel formats and translates messages into a network-specific form and delivers them to an endpoint on the server.

Channels also impose an order to the flow of messages sent to the server and the order of corresponding responses. Order is important to ensure that interactions between the client and server occur in a consistent, predictable fashion.

Channels communicate with Java-based endpoints on the server. An endpoint unmarshals messages in a protocol-specific manner and then passes the messages in generic Java form to the message broker. The message broker determines where to send messages, and routes them to the appropriate service destination.
![blazeds_channel_endpoint_intro](http://livedocs.adobe.com/blazeds/1/blazeds_devguide/images/blazeds_channel_endpoint_intro.png)

####Channel types

BlazeDS includes several types of channels, including standard and secure Action Message Format (AMF) channels and HTTP (AMFX) channels. AMF and HTTP channels support non-polling request-response patterns and client polling patterns to simulate real-time messaging. The streaming AMF and HTTP channels provide true data streaming for real-time messaging.

###BlazeDS summary of features

The following table summarizes some of the main features of BlazeDS:

<table width='100%'>
<tr>
<th>
Feature
</th>
<th>
Description
</th>
</tr>

<tr>
<td>
Proxy service
</td>
<td>
Enables communication between clients and domains that they cannot access directly, due to security restrictions, allowing you to integrate multiple services with a single application. By using the Proxy Service, you do not have to configure a separate web application to work with web services or HTTP services.
</td>
</tr>

<tr>
<td>
Publish and subscribe messaging
</td>
<td>
Provides a messaging infrastructure that integrates with existing messaging systems such as JMS. This service enables messages to be exchanged in real time between browser clients and the server. It allows Flex clients to publish and subscribe to message topics with the same reliability, scalability, and overall quality of service as traditional thick client applications. 
</td>
</tr>

<tr>
<td>
Software clustering
</td>
<td>
Handles failover when using stateful services to ensure that Flex applications continue running in the event of server failure. The more common form of clustering using load balancers, usually in the form of hardware, is supported without any feature implementation.
</td>
</tr>
</table>
