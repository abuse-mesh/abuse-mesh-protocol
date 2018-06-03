# Abuse Mesh

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

In Abuse Mesh a blacklist is compiled based on a few faactors: report amount, report quality and node thresholds.
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
The abuse mesh protocol uses the HTTP protocol as a base, like a REST API diffrent request types have diffrent HTTP request methods. 
Request data is transfered in the request body in a JSON format. 
A response code will tell the state of a response using the default HTTP code meanings.
A response body contains requested data.
The request and response headers can be used to transfer metadata about a client, server or the state of a connection
All HTTP requests should be directed to a uri with a base path of /mesh/ so as to allow for a GUI or other application on other parts of the webserver


### Node config request

### Connecting to the mesh
In order to receve reports from a neighbor the node has to start a connection to this node. Depending opon the configuration of the server all nodes are allowed to connect.
However a node can also restrict access to a specific group of nodes. This can be done any way two nodes can agree to authenticate and is based on the implementation.
A few notable ideas for authentication are: OAuth, Pre shared keys and authentication based on source IP.

1. The client sends a CONNECT request to the server on path /mesh/connect
2. The server determens if an client is allowed to connect (either based on caracteristics or based on authentication)
3. The server will send a 202(Accepted) or a 401(Unauthorized) response code depending opon the desision the server made in step #2 

### Disconnection from the mesh

### Synchronising report tables

### Subscribe to changes

### Unsubscribe to changes

### Sending a batch of new reports

### Sending a batch of delisted reports

### Delist request

### Verified delist request propagation
