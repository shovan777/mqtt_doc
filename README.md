links:
https://www.hivemq.com/mqtt-essentials/
https://www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt
use mqtt-sn to connect over udp instead of tcp




## Intro
-MQTT is a Client Server publish/subscribe messaging transport protocol.
-It is light weight, open, simple, and designed so as to be easy to implement.
-MQTT officially does not have an acronym anymore, it’s just MQTT.
-previously MQTT = MQ Telemetry Transport
- Pub/Sub decouples a client, who is sending a particular message (called publisher) from another client (or more clients), who is receiving the message (called subscriber). 
-This means that the publisher and subscriber don’t know about the existence of one another. There is a third component, called broker, which is known by both the publisher and subscriber, which filters all incoming messages and distributes them accordingly. 

Space decoupling: Publisher and subscriber do not need to know each other (by ip address and port for example)

Time decoupling: Publisher and subscriber do not need to run at the same time.

Synchronization decoupling: Operations on both components are not halted during publish or receiving


## Publish/subscribe basics
- pub/sub causes following decoupling:
	- Space decoupling
	- Time decoupling
	- Synchronization decoupling

- message filtering: done by broker so that each subscriber only recieves messages it intends to listen to
1. subject-based filtering: based on topic. Topics are in general strings with an hierarchical structure
2. content-based filtering: based on content filter language
3. type-based filtering: based on type/class of message

- challenges in pub/sub due to decoupling:
	- subscriber should know the structure of published data beforehand
	- both must be aware of the topics
	- publisher does not know if the messages are being listened toCl
- the broker is able to store messages for clients that are not online. (This requires two conditions: client has connected once and its session is persistent and it has subscribed to a topic with Quality of Service greater than 0)

- MQTT uses subject based filtering and has all types of decoupling but coupling can be achieved if required
- So each message contains a topic, which the broker uses to find out, if a subscribing client will receive the message or not
-  MQTT has the quality of service (QoS) levels. It is easily possible to specify that a message gets successfully delivered from the client to the broker or from the broker to a client.

- Message queue vs MQTT
	- queue stores messages until consumed by one client and otherwise they are stuck so, each message must always be processed unlike in mqtt
	- each message is listened to by only one client but in mqtt all clients subscribe to the topic will get the message

## Client, broker and connection establishment
- topic is a hierarchical structured string, which is used for message filtering and routing

### Client
- MQTT client can be both a publisher & subscriber at the same time
- A MQTT client is any device from a micro controller up to a full fledged server, that has a MQTT library running and is connecting to an MQTT broker over any kind of network. 

### Broker
- a broker can handle up to thousands of concurrently connected MQTT clients. 
- The broker is primarily responsible for receiving all messages, filtering them, decide who is interested in it and then sending the message to all subscribed clients.
- holds session for all persisted clients
- authentication and authorization of clients
- broker is directly exposed to the internet
- it is highly scalable, integratable into backend systems, easy to monitor and of course failure-resistant.

### MQTT connection
- mqtt is on top of tcp/ip stack
- The connection is initiated through a client sending a CONNECT message to the broker. The broker response with a CONNACK and status code
- Once the connection is established, the broker will keep it open as long as the client doesn’t send a disconnect command or it looses the connection.
- if the CONNECT message is malformed (according to the MQTT spec) or it takes too long from opening a network socket to sending it, the broker will close the connection.
- the connection will be kept open to allow sending and receiving message bidirectional after the initial CONNECT.

#### Connect Message
-contains:
clientId*
cleanSession*
username
password
lastWillTopic
lastWillQos
lastWillMessage
lastWillRetain
keepAlive*

- * means compulsory all else is optional

- clientId
	- client identifier
	- unique per broker
	- identify client and status of client
- it is also possible to send an empty ClientId, which results in a connection without any state

- Clean Session
	- clean session flag indicates the broker, whether the client wants to establish a persistent session or not
	- A persistent session (CleanSession is false) means, that the broker will store all subscriptions for the client and also all missed messages, when subscribing with Quality of Service (QoS) 1 or 2.
	- If clean session is set to true, the broker won’t store anything for the client and will also purge all information from a previous persistent session.

- Will message
	- allows to notify other clients, when a client disconnects ungracefully. 

- Keep Alive
	- keep alive is a time interval, the clients commits to by sending regular PING Request messages to the broker. The broker response with PING Response and this mechanism will allow both sides to determine if the other one is still alive and reachable. 

#### CONNACK message
Session Present Flag
- session present flag indicate, whether the broker already has a persistent session of the client from previous interactions. 

Acknowledge Flag (returnCode)
-  It signals the client, if the connection attempt was successful and otherwise what the issue is.

## MQTT publish, subscribe and unsubscribe
### Publish
-  each message must contain a topic, which will be used by the broker to forward the message to interested clients. Each message typically has a payload which contains the actual data to transmit in byte format.
- mqtt is data agnostic
- contains:
	packetId
	topicName
	qos
	retainFlag (0 if QOS is zero)
	payload
	dupFlag

#### Topic Name
- A simple string, which is hierarchically structured with forward slashes as delimiters. 
#### Retain flag
- This flag determines if the message will be saved by the broker for the specified topic as last known good value. 
#### DUP flag (Duplicate flag)
- The duplicate flag indicates, that this message is a duplicate and is resent because the other end didn’t acknowledge the original message

### Subscribe
- A subscribe message is pretty simple, it just contains a unique packet identifier and a list of subscriptions.
- contains:
	packetId
	qos1
	topic1
	qos2
	topic2
	...
-  If there are overlapping subscriptions for one client, the highest QoS level for that topic wins and will be used by the broker for delivering the message.

### Suback
- contains:
	packetId
	returnCode 1
	returnCode 2
	...

### Unsubscribe
- The counterpart of the SUBSCRIBE message is the UNSUBSCRIBE message which deletes existing subscriptions of a client on the broker. 
- contains:
	packetId
	topic1
	topic2
	...

### Unsuback
- broker will acknowledge the request to unsubscribe with the UNSUBACK message.



## MQTT topics and best practices
- A topic is a UTF-8 string, which is used by the broker to filter messages for each connected client. A topic consists of one or more topic levels. Each topic level is separated by a forward slash (topic level separator).
- eg: myhome/groundfloor/livingroom/temperature
- must have atleast 1 character
- can contain spaces
- topics are case-sensitive
- forward slash alone is a valid topic

### Wildcards
- When a client subscribes to a topic it can use the exact topic the message was published to or it can subscribe to more topics at once by using wildcards.
- can only be used while subscribing and not while publishing
- types:

#### single level: +
- single level wildcard is a substitute for one topic level.
- written as a plus symbol
- eg: myhome/groundfloor/+/temperature
- valid/invalid:
	myhome/groundfloor/kitchen/temperature
	myhome/groundfloor/baba/bigi/temperature

#### Multi level: #
- covers an arbitrary number of topic levels.
- it is required that the multi level wildcard is always the last character in the topic and it is preceded by a forward slash.
- eg: myhome/groundfloor/#
- valid/invalid:
	myhome/groundfloor/livingroom/temperature
	myhome/firstfloor/kitchen/temperature

### Topics beginning with $
- Each topic, which starts with a $-symbol will be treated specially and is for example not part of the subscription when subscribing to #
- reserved for internal statistics of the MQTT broker
- no clear official standarization
- common practice use $SYS/ 

### Best Practices
1. Don't use a leading forward slash
- It is allowed to use a leading forward slash in MQTT, for example /myhome/groundfloor/livingroom. But that introduces a unnecessary topic level with a zero character at the front.
2. Don't use spaces in a topic
3. keep the topic short and concise
- Each topic will be included in every message it is used in, small topic saves bytes
4. Use only ASCII characters, avoid non printable characters
5. Embed a unique identifire or the ClientId into the topic
6. Don't subscribe to # 
- instead add an extension to MQTT broker to add a asynchronous routine to process each incoming message and persist it to a database.
7. Don't forget extensibility
- For example when your smart home solution is extended by some new sensors, it should be possible to add these to your topic tree without changing the whole topic hierarchy.
8. Use specific topics, instead of general ones
- if you have three sensors in your living room, you should use topics myhome/livingroom/temperature, myhome/livingroom/brightness and myhome/livingroom/humidity, instead of sending all values over myhome/livingroom. Also this enables you to use other MQTT features like retained message



## QOS: Quality of Service 0, 1, 2
- 0: at most once
- 1: at least once
- 2: exactly once
- The QoS level for publishing client to broker is depending on the QoS level the client sets for the particular message
- When the broker transfers a message to a subscribing client it uses the QoS of the subscription made by the client earlier.
- QoS guarantees can get downgraded for a particular receiving client if subscribed with a lower QoS.

### QOS 0
-  A message won’t be acknowledged by the receiver or stored and redelivered by the sender. This is often called “fire and forget” and provides the same guarantee as the underlying TCP protocol.

### QOS 1
- When using QoS level 1, it is guaranteed that a message will be delivered at least once to the receiver. But the message can also be delivered more than once.
- The sender will store the message until it gets an acknowledgement in form of a PUBACK command message from the receiver.

### QOS 2
- guarantees that each message is received only once by the counterpart.
- safest and also the slowest quality of service level.

### Best practices of using QOS levels
#### Using QoS 0 when 
- You have a complete or almost stable connection between sender and receiver. A classic use case is when connecting a test client or a front end application to a MQTT broker over a wired connection. 

- All messages sent with QoS 1 and 2 will also be queued for offline clients, until they are available again. But queuing is only happening, if the client has a persistent session.

## Persistent Session and Queuing Messages
### Persistent Session
- on reconnect client needs to resubscribe the topics
- this is expected behaviour without persistent session
- persistent session saves all information relevant for the client on the broker. The session is identified by the clientId provided by the client on connection establishment 

#### what are stored in the session
- Existence of a session, even if there are no subscriptions
- All subscriptions
- All messages in a Quality of Service (QoS) 1 or 2 flow, which are not confirmed by the client
- All new QoS 1 or 2 messages, which the client missed while it was offlne
- All received QoS 2 messages, which are not yet confirmed to the client
 
- even if the client is offline all the above will be stored by the broker and are available right after the client reconnects.

#### start/end persistent session
- persistent session is requested by client on connection establishment by setting cleanSession flag of connect message to false
- When clean session is set to false, a persistent session is created and it will be preserved until the client requests a clean session again
- If there is already a session available then it is used and queued messages will be delivered to the client if available.
- the CONNACK message from the broker contains the session present flag, which indicates to the client if there is a session available on the broker

#### Persistent session on Client side
- Similar to the broker, each MQTT client must store a persistent session too. 
- i.e. it must store:
	- All messages in a QoS 1 or 2 flow, which are not confirmed by the broker
	- All received QoS 2 messages, which are not yet confirmed to the broker

### Best Practices
#### Persistent Session
-    A client must get all messages from a certain topic, even if it is offline. The broker should queue the messages for the client and deliver them as soon as the client is online again.
-    A client has limited resources and the broker should hold its subscription, so the communication can be restored quickly after it got interrupted.
-    The client should resume all QoS 1 and 2 publish messages after a reconnect.

#### Clean Session
-    A client is not subscribing, but only publishing messages to topics. It doesn’t need any session information to be stored on the broker and publishing messages with QoS 1 and 2 should not be retried.
-    A client should explicitly not get messages for the time it is offline.

#### limit of persistent session
- if a client does not come online for a long time? The constraint for storing messages is often the memory limit of the operating system. 
- there is no standard defined in mqtt for this and is rather handled by the broker



## Retained Messages
- A retained message is a normal MQTT message with the retained flag set to true. The broker will store the last retained message and the corresponding QoS for that topic 
- as soon as client subscribe to a topic they will recieve the retained message
- there is only one retained message for each topic
- client knows it has received a retained message if retained flag is set true

#### Sending a retained message
- set the retained flag of a MQTT publish message to true
#### delete a retained message
- send a retained message with a zero byte payload on that topic where the previous retained message should be deleted
- deleting is not necessary, because each new retained message will overwrite the last one.
- Without retained messages new subscribers are kept in the dark between publish intervals. So using retained messages helps to provide the last good value to a connecting client immediately.
- for eg: a device sending gps coordinates in long intervals



## Last Will and Testament (LWT)
- mqtt is often used in unreliable networks
- i.e. some clients may disconnect ungracefully ( without mqtt DISCONNECT message)

### LWT message
- Each client can specify its last will message (a normal MQTT message with topic, retained flag, QoS and payload) when connecting to a broker.
-  If the client disconnect abruptly, the broker sends the message to all subscribed clients on the topic
- stored LWT message will be discarded if a client disconnects gracefully by sending a DISCONNECT message.

#### DISCONNECT message
- it contains nothing i.e empty payload

#### Specifying the LWT message
- The LWT message can be specified by each client as part of the CONNECT message

#### Conditions for broker to send LWT
-    An I/O error or network failure is detected by the server.
-    The client fails to communicate within the Keep Alive time.
-    The client closes the network connection without sending a DISCONNECT packet first.
-    The server closes the network connection because of a protocol error.


## Keep Alive and Client Take-over
### Problem of half-open TCP connections
- the communicating parties gets out of sync with the other, often due to a crash of one side or because of transmission errors.
- still functioning end is not notified about the failure of the other side and is still trying to send messages and wait for acknowledgements.
- Although TCP/IP in theory notifies you when a socket breaks, in practice, particularly on things like mobile and satellite links, which often “fake” TCP over the air and put headers back on at each end, it’s quite possible for a TCP session to “black hole”, i.e. it appears to be open still, but in fact is just dumping anything you write to it onto the floor.

### MQTT Keep Alive
- client specifies the longest period of time specified during connection establishment which broker and client can endure without sending a message
- It is the responsibility of the Client to ensure that the interval between Control Packets being sent does not exceed the Keep Alive value. In the absence of sending any other Control Packets, the Client MUST send a PINGREQ Packet.
- The broker must disconnect a client, which doesn’t send PINGREQ or any other message in one and a half times of the keep alive interval.

#### PING REQ
- contains no payload
- The PINGREQ is sent by the client and indicates to the broker that the client is still alive, even if it hasn’t send any other packets

#### PING RESP
- contains no payload
- When receiving a PINGREQ the broker must reply with a PINGRESP packet to indicate its availability to the client. 

#### some facts
- If the broker doesn’t receive a PINGREQ or any other packet from a particular client, it will close the connection and send out the last will and testament message (if the client had specified one).
- The MQTT client is responsible of setting the right keep alive value. For example, it can adapt the interval to its current signal strength.
- The maximum keep alive is 18h 12min 15 sec.
- If the keep alive interval is set to 0, the keep alive mechanism is deactivated.

### Client Take-Over
- incase, broker holds a half-open connection, The broker will close the previous connection to the same client (determined by the same client identifier) and establishes the connection with the newly connected client.
- This behavior makes sure that half-open connection won’t stand in the way of a new connection establishment of the same client.
