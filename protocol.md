# Abuse Mesh

## Motivation
Abuse Mesh was created as an alternative to the current abuse reporting and blacklisting systems.

The situation as of the creation of this document is quite bad. Almost all abuse reports are being sent of email, which needs to be processed by a human. At large scale this becomes a tough job. Attempts have been made to somewhat improve the process by parsing the emails, and processing the reports automatically. However this often times doesn't work since most emails don't adhere to a standard.

A similar yet different situation is the blacklisting systems which are around at the moment. Often times the big online blacklists are managed by secretive organizations which create these blacklists based on there own findings or opinions. Often they don't state any proof of what they clame to have found.

These blacklist are published in .txt, .csv or .xls files which don't follow any standard. The result is that every blacklist has a diffrent format which needs special attention to parse and use.

Once a IP owner(often ISP's or other service providers) has dealt with the issue a delist request has to be emailed to the organization which manages the blacklist. Sometimes it can take days or weeks before a listing is removed, if it gets removed at all.

Abuse Mesh aims to resolve all of the issues listed above. By creating distributed network without a single authority we can collectively determin which IP's, websites and ASN's are rotten apples. The collection, distribution and aggregation of abuse reports and blacklists can thus be done in a standardized manner which can easily interact with automated systems.

## Definitions of Terms Used

* node - a node is server that runs an application that implements the Abuse Mesh protocol
* neighbor - a neighbor is a node that directly communicates with your own node
* client - a node that receves or wants to receve reports from a neighbor
* server - a node that sends or wants to send reports to a neighbor
* report - an abuse report in the abuse mesh network
* batch - a collection of reports, either a compleat table or a spesific collection that is transfered in the network
* table - a collection of all reports a node knows of
* creator - the node that created a report
* suspect - the domain name, IP address or IP range and the ASN that is accused of abusing the internet  
* blacklist - a list of reports that is deemed convident enough to be used to undertake action against a suspect

* AS(N) - autonomous system (number)
* PGP - Pretty Good Privacy, a method for encryption, decryption and digital signing of messages
* UUID - Universally unique identifier
* RIR - Regional Internet Registry

## Concepts and constructs

The abuse mesh protocol uses a view concepts and constructs to determine the working of the mesh network as well as the quality of reports, blacklists and nodes.

### blacklists

A blacklist normally is a list of IP addresses and ranges that are deemed abusive by an authority, most of the time this is the same authority that hosts and distributes this list.
Since Abuse Mesh is decentralized and doesn't have a managing authority we need some other way of making blacklists.

In Abuse Mesh a blacklist is compiled based on a few factors: report amount, report quality and node thresholds.
The precess works as follows:

1. A node collects all reports it want's to use based on it's import policy
2. All reports are grouped based opon suspect
3. The node makes sure no duplicate reports exist from the same creator
4. For each creator that reported a suspect there report quality is summed, this sum is the report score for this suspect
5. Every suspect with a report score higher then the threshold configured in the node is put on the blacklist

### report quality

The report quality of a creator is nessesery to insure the quality of reports. If we were to omit report quality in the blacklist creation process.
A entity like a corporate rival or state government could deny service to a suspect by copying reports and boosing report rate.

To calculate report quality the PageRank method is used, but instead of one page linking to another a link is one node recieving reports from another node.
To start, a node will "crawl" the mesh and index all connections in the network for as long as it is willing to crawl or as found all connections
All connections are fed in a PageRank implementation and the result saved for later usage

### Delisting

If a suspect claims to have solved the abuse that was reported he can send a delist message. 
This delist message is distributed accros the network to the creators of the report.
Once a creator receves a delist message the node will start a countdown with a configurable time. If a new event refreshes the report within the countdown the report stays
However if the countdown expires within the countdown the report is removed from the table and synced to the network

Checks need to be in place to prevent internet abusers from delisting bad ip's. That is why a delist message has to be accepted by a node with a high report quality.
A node may only accept a delist message if the suspect can prove that he is the owner of the suspected IP address / IP range.
The exact method used is based on the implementation. A few notable ideas are: 
* requesting for a token sent over email to an address listed in the whois info
* manual validation
* validation using a pgp or x509 public keys from a RIR (for instance from a key-cert object in the RIPE db)
* whitelisting of previously validated node's

## Protocol

The Abuse Mesh protocol uses HTTP(HTTP over TLS and HTTP/2) as a transport protocol and MessagePack for data encoding. There are several reasons why HTTP was chosen as the transport protocol which are as follows:

* HTTP has wide platform support and is already used in REST, JSON-RPC and GRPC APIs which makes it easer to implement on multiple platforms.
* There is already a lot of software and infrastructure in place to loadbalance and distribute HTTP traffic which means that scaling up a network of Abuse Mesh nodes is easy to do.
* HTTP has authentication standards, encryption, metadata(headers) and request methods which all can be reused.
* HTTP can be cached and used in CDN's which is useful for blacklists and so on.

Every of these points would be problematic or at least a lot harder when using a binary protocol over TCP. As for the data encoding the choice to use MessagePack was made because it is a faster and more compact version of JSON. It has built in support for more data structures including binary data.

### Versioning

This is the first iteration of the document, but since the protocol is expected to evolve in ways that are not yet known or imagined at this point the protocol should be ready for versioning from the start. The protocol version will used MAJOR.MINOR.PATCH version numbering. Where major version increments break compatibility, minor increments add functionality in a way that is backwards compatible and patch increments are bug or security fixes which are backwards compatible within the major version.

### URL

Because the Abuse Mesh protocol uses HTTP it is possible to host another web application on the same domain, this can be useful for potential web GUI's and so on. To avoid URL collisions with other applications all paths in the Abuse Mesh protocol must be prefixed with /mesh. Because only major version may change the required part of a signature of a request all we can use the major version number as part of the URL to allow users to upgrade to a new version gracefully and run multiple major version at the same time. So for version 1 of the protocol all paths start with /mesh/v1 and for a second major version the path prefix would become /mesh/v2.

Just like with REST API's the URL indicates a resource, which is generated at runtime or stored and retrieved. Parts of the URL are variables for accessing certain resources in a REST API manner.

### Request methods

Since the Abuse Mesh protocol uses HTTP the HTTP request methods are also used to communicate intent for requests. Since the Abuse Mesh protocol employed a PUB/SUB model the Abuse Mesh protocol uses some exotic request methods. The request methods used and there meanings are as follows:

* GET    - Used to 'get' dat from the server, has no request body and the server may never insert, update or delete data on a GET request
* POST   - Prompt the server to insert new data or execute a function which does so (like a synchronization action). May contain a request body.
* PUT    - Replaces a resource on the server with a resource in the request body.
* PATCH  - Partially updates a resource, only the properties which should be overwritten should be sent in the request body
* HEAD   - Is used when a client only want metadata about a resource to check if it has be updated and/or if it exits
* LINK   - Is used to signal to a server that a node wants to subscribe to updates of a resource via the PUB/SUB mechanism
* UNLINK - Is used to signal to a server that a node wants to unsubscribe from updates of a resource via the PUB/SUB mechanism

### Response codes

As per the HTTP protocol the server will respond to every request with a response, in which is a response/status code. In general the same meaning is retained as specified in the HTTP RFC's. The following codes are commonly used or are custom so there meanings are written down for clarity sakes and are as follows:

* 200 - There were no errors while processing this request
* 201 - A new resource is created
* 202 - A change has been triggered on the server but runs asynchronously (the client may need to poll to get status updates or may receive a status update via the PUB/SUB mechanism)
* 400 - The request that was made by the client doesn't follow this specification and can't be processed
* 401 - The client is not authenticated but should be to view or modify this resource
* 403 - The client is not allowed to view or modify a resource
* 404 - A URL path doesn't exist or a resource doesn't exist(The response body must clarify)
* 500 - The server has encountered a error from which it can't recover so can't complete the request(almost always a bug)
* 503 - The server refuses to process the request because it is overloaded (Most likely the client reached a rate limit)

### PUB/SUB mechanism

Because of the nature of abuse reports it is desireable to be able to subscribe to a server from which a node wants to receive updates about new abuse report, delisting requests or any other data that may change frequently. If implemented correctly this means that data updates can propagate through the mesh network at a fast rate which can be crucial when sites need to quickly be put on a blacklist or delisted from one.

The PUB/SUB mechanism has a few requirements which are as follows:

* It must be fault tolerant (network issue can and will happen)
* It must handle ungraceful "disconnects"
* It must be mindful of network congestion and transmission rates

#### Fault tolerance

The event publisher will for every subscribed node record a autoincrementing identifier which for every published event will increment. This id is sent along with the event. The subscriber should keep record of which ids have been received. If for some reason a event publisher was not able to deliver the event to the subscriber the subscriber will notice that it misses one or more ids in the sequence. At that point the subscriber can request the missing events from the publisher or resyncronize the whole resource. If the publisher no longer has record of the event the subscriber must resyncronize the resource.

#### Ungraceful "disconnects"

The event subscriber may at some point go offline, lose state or be unreachable for extended periods of time. In failure situaltion like that the event publisher will not recieve a UNLINK request and will keep trying to connect to the subscriber. This causes network timeouts and I/O blocks which are undesirable for performance. After a X amount of failed connection attempts on any protocol layer the publisher must mark the subscriber as nonresponsive. If a subscriber is nonresponsive the rate of event updates will be reduced until the subscriber regains the ability to receive the event in which case the subscriber can attempt to recover. If a subscriber doesn't respond within a X amount of time the server must unsubscribe the node.

#### Network congestion and transmission rates

If a node has a lot of subscribers or it publishes to a lot of nodes the amount of transmitted data may become very high. To make the transmission of updates as efficient as possible the updates will be sent in bulk. When the amount of changes exceeds a threshold a update should be sent or else a update should be sent after a certain amount of time. These values should be configurable by the nodes. For example a node could be set to send a event-bulk if there are 512 or more events waiting to be published or if it has been one minute since the last update was published. These rates should be nagotiated between the publisher and subscriber.

#### Subscribing to a node

If a resource on the server supports update publishing it will have a endpoint which accepts LINK and UNLINK request methods.

1. The client makes a LINK request for a specific resource to the server
2. The server will check if the client is authenticated and authorized to subscribe to a resource (a 401 or 403 state is send back if this step fails)
3. The server will add the client to a list of subscriber along with a unique identifier for the subscription
4. and return the complete resource to the client in the response body along with the subscription identifier
5. The client will replace the local resource with the one the server sent to it and add the server and the subscription identifier to a list of nodes the client is subscribed to

#### Publishing to a node

If the a resource in the publisher is updated the publisher will get a list of nodes which are subscribed to that resource and add the event to the subscriber queue. When the queue reaches the transmission threshold the following steps happen:

//TODO tell about endpoints

1. The publisher increments the update id
2. The publisher saves the event-bulk in the history
3. The publisher sends the events to the subscriber (If this fails we stop here)
4. The subscriber checks the authentication and authorization of the publisher (if the publisher is unauthorized the publisher will unsubscribe the node since it will never be able to complete a request)
5. The subscriber checks the subscription id to see if it matches (If the ids are different the state if the subscriber and publisher are unequal and will have to do a full resyncronization)
6. The subscriber processes all the events and replies with a 200
7. If the publisher gets a 200 response the saved event-bulk is purged from its history, if a error at any point has occurred the history is kept for a configurable amount of time

#### Unsubscribing from a node

//TODO