# AbuseMesh

## Motivation
AbuseMesh was created as an alternative to the current abuse reporting and blacklisting systems.

The situation as of the creation of this document is quite bad. Almost all abuse reports are being sent of email, which needs to be processed by a human. At large scale this becomes a tough job. Attempts have been made to somewhat improve the process by parsing the emails, and processing the reports automatically. However this often times doesn't work since most emails don't adhere to a standard.

A similar yet different situation is the blacklisting systems which are around at the moment. Often times the big online blacklists are managed by secretive organizations which create these blacklists based on there own findings or opinions. Often they don't state any proof of what they claim to have found.

These blacklist are published in .txt, .csv or .xls files which don't follow any standard. The result is that every blacklist has a different format which needs special attention to parse and use.

Once a IP owner(often ISP's or other service providers) has dealt with the issue a delist request has to be emailed to the organization which manages the blacklist. Sometimes it can take days or weeks before a listing is removed, if it gets removed at all.

AbuseMesh aims to resolve all of the issues listed above. By creating distributed network without a single authority we can collectively determin which IP's, websites and ASN's are rotten apples. The collection, distribution and aggregation of abuse reports and blacklists can thus be done in a standardized manner which can easily interact with automated systems.

## Definitions of Terms Used

* 'node' - a node is server that runs an application that implements the AbuseMesh protocol
* 'neighbor' - a neighbor is a node that directly communicates with your own node
* 'client' - a node that receves or wants to receve reports from a neighbor
* 'server' - a node that sends or wants to send reports to a neighbor
* 'report' - an abuse report in the AbuseMesh network
* 'table' - a collection of all reports a node knows of
* 'creator' - the node that created a report
* 'suspect' - the domain name, IP address or IP range and the ASN that is accused of abusing the internet  

* AS(N) - autonomous system (number)
* PGP - Pretty Good Privacy, a method for encryption, decryption and digital signing of messages
* UUID - Universally unique identifier
* RIR - Regional Internet Registry

## Protocol

The protocol definition consists of two parts: The wire protocol defined in Protocol buffers API description language and this document which describes the expected behavior of a 'node'. These parts are versioned together and may reference to each other.

### Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

### Versioning

The AbuseMesh protocol is versioned using [semantic versioning 2.0.0](https://semver.org/spec/v2.0.0.html).

