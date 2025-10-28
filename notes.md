# Discussion Questions

Questions are organized by topics.

## Resource Discovery

Does DHCP work internet-wide?

* No. Recall that DHCP (Dynamic Host Configuration Protocol) is used to identify all resources on the network by having everyone broadcast who they are, then assigning IPs to them. If DHCP was to work internet-wide, we'd require a list of every device connected to the internet, which is too many to scale practically.

How does DNS start? How do we make a new DNS resolver?

* Hardcode the IP addresses of the 13 root servers into our resolver and query them, which will point our DNS resolver to the correct location and IP address of any domain it has to resolve.

DNS gives us location transparency. Why is location transparency great?

* The internet becomes easier to use, because we don't have to remember the exact IP address of every webpage we'd like to regularly visit.
* Hosts can be moved without the client needing to know.
* Hide complexity: the client does not know exactly how many machines it was talking to.

Is the location 100% hidden?

* No. At some point, the domain must be resolved to an IP address.

Are DNS entries globally unique?

* In the context of the course, yes. The DNS hierarchy distributes what domain resolves to what address and via which DNS server.

How do we make DNS faster?

* Caching. Once a resolver finds an IP address, it can store it.
* However, the host may move, rendering our cached resolution invalid. One way to circumvent this is implementing Time To Live (TTL): each cache entry is only valid for a fixed amount of time. This, however, isn't secure, and is vulnerable to cache poisoning and DNS stealing attacks.

Why do we have different entities administrate different levels of DNS?

* Distribute compute (ha-ha) and responsibilities. Prevent single-point-of-failure.

How does the global DNS system ensure that things are unique? What if two people register the same domain at the same time, or 2 DNS servers announce different changes?

* No one-size-fit-all solution. Possible solutions include mandating consensus between DNS servers.

What is concurrency? What is consensus?

* Concurrency is when actions are being performed or events are happening simultaneously.
* Consensus is when all machines in the network agree on the state of the system. Not an easy problem to solve.

Why do we need ports?

* Recall that a port is a logical construct on a machine that specifies where some communication should be received. Allow communication from one machine to multiple different machines at the same time using a relatively standarized format.

What are the limitations of ports?

* Ports are limited in number (free ports: 1024-65535). In practice, not really felt.

Can DNS resolve *services*? Why not? How does this impact transparency?

* No. DNS only resolves the IP address, but not a specific port; a service running on another port in a server may not be resolved to.
* To access a service listening on a given port, we have to know what that specific port is, reducing location transparency.

How can we solve addressing services? How do names get configured/managed? What if a service needs to move *while maintaining transparency*?

* One option is to design a name server/hostpost server that resolves not only IPs, but also ports of defined services.
* Services should register themselves with the hostpost server, so it knows on which port is the service located.
* An intermediary (like our name/hostpost server) should always be tracking the service hosts that could move, though this still requires the services to announce themselves to the intermediary. When a client wishes to access a service, it will need to go through this intermediary which has knowledge of the moving service's location, maintaining location transparency and relocation transparency from the client's perspective.

How do we know where the name server is? At some point, do you *have to have* well known services?

* You need to know which port it's on to connect to it. At some point, needing well known services is required. Well-known ports help, e.g. the `ssh` protocol always connect to port 22.

Are "/" paths?

* Yes. Always. Everything is a path. Everything is a file.

## Message Passing

What is message passing?

* Sending any message. Note that by sending messages, you are sharing state.

What are issues with message passing?

* Latency: no guarantee on how quickly a message arrives after being sent.
* A message may not arrive at all.
* Either host may drop while sending, or having sent, the message, before it is received.
* The message may be garbled or incomplete upon arrival.
* Messages may arrive out of order.

What is state?

* Any data stored by a host - client, or server. Specifically, state is data that influences further communication in the system. Note that not all data is state.

How does an SSH server know who you are?

* Link state - the status of each connection. The server stores who is connected on each port.

What are examples of stateless servers?

* Basic HTTP site (without dynamic requests)
* FTP (File Transfer Protocol), sometimes
* Relays
* IRC only holds link state (in theory)
* Root DNS servers (mostly)

What are examples of stateful servers? What state does it hold?

* Any dynamic website, for example storing information about users
* SSH

How much state does the client need to get appropriate data?

* It depends. Rule of thumb is as little as possible, but there are some drawbacks to this.

How do we share state?

* Sending state as part of the message.

When fetching new threads/comments/articles, what is the thick *client* solution? What is the thin *client* solution? Can one be more effective at saving bandwith?

* Thick client gets all threads/comments/articles from server, and only displays new ones.
* Thin client only gets new threads/comments/articles from the server.
* In this case, the thin client uses less communication.

How do we establish and guarantee connections?

* Server should have DNS or well-known IP address so clients can establish connections to server. Clients can then periodically check if connection is still up, and re-establish if necessary.

How would "offline" work with thick clients? How does Dropbox do it? Git?

* State will be desynchronized. There are collisions that need to be resolved once the client comes back online.
* Possible strategies include taking the most recent, having a fixed priority on who to trust, recording all changes and doing them in order, and letting the user choose (this is what Git does; if the remote repository pulled disagrees with your local code, it's up to you to resolve them and inform the server of the "right" resolution. Not sure about Dropbox - author's notes).

How do we sync state? How do we preserve state under failure? How could a system be more resilient to failures?

* No real solution, depends. Syncing between thick clients and stateful servers is complicated.
* Generally, state should only be processed where it is stored. Consider every participating host and who needs to push or pull what state.

Why would we use a relay versus a buffer?

* Buffer: The receiver needs to actually get the message, or the host receiving the messages may be busy (maybe a server with many clients).
* Relay: Sending messages from a server to many clients (maybe because there are too many messages to store at one server), or if there are many messages and not all of them are necessary.

## Practical Communication

What encodings are there? What do sockets send?

* Sockets only directly send binary.
* The other, intermediary, more human-readable encoding is text. Some standardized encodings are ASCII and UTF-8. Note that JSON, XML, HTML,...are all considered text encodings.

Why would we encode directly to binary? What are some disadvantages of binary?

* Some things are hard to translate to text, such as images, audio, and video. Binary representations can also be more compressed, saving bandwidth.
* Binary may be problematic to decode e.g. mismatched endianess between sender and receiver.

How do we send an object on a socket?

* Flatten/serialize/pickle/marshall the object, for example using JSON, then encode to binary.
* Alternatively, encode to binary directly.

With flattening, what can we not encode? How can we fix this?

* Circular linked list. File-like objects, such as sockets themselves.
* Solutions include regenerating the pointers somehow, usually letting each object define its own `encode` method.

Why do most protocols use TCP instead of UDP? Why does NTP use UDP?

* TCP guarantees message delivery and can get reliable responses.
* NTP (Network Time Protocol) requires constant polling for time synchronization. UDP is faster and so is used, but needs to be more precise with it.

Why are reflection attacks less effective on TCP?

* Recall that reflection attacks misuse UDP's lack of connection, allowing any return address to be written in the message. TCP requires acknowledgement by the server of the client to send messages, so addresses are harder to spoof.

When would UDP multicast be useful? What are the limitations?

* Announcing you're on a network, chatrooms/group calls/group chats, or news/post feeds. Generally, when everyone should be on the same frame/block of bytes.
* The message reception may get lossy due to UDP. It's hard to roll back and re-ask for the message.

What are the pros/cons of blocking?

* Pros:
  * Will respond to/process the message as soon as it's available.
  * Know for sure message was delivered before continuing.
  * Good for listening on a port.
* Cons:
  * The message may never arrives, suspending the host from receiving new connections.
  * May wish to monitor multiple ports and listen for new connections concurrently (multithreading can help solve this).

What are the pros/cons of polling?

* Pros:
  * Other tasks can be done while waiting.
* Cons:
  * Annoying if a message is needed to continue.
* Sometimes, polling is the only option due to device's capabilities.

Would a modern server ever do polling?

* Yes. For example, workers polling for jobs from a work queue.

Is there a third option?

* Block with a timeout.

We have a chathopper server that echoes messages sent to it. It has a timeout, why?

* To prevent a connected client that doesn't send any message to keep the server blocked forever.

How do we communicate with multiple clients on TCP connections?

* Python's `select` takes non-blocking file-like objects and blocks on their behalf until something happens on any of them.
* Multithreading

Suppose you're designing a system that needs to transfer a lot of data from one machine to another, and it needs to happen quickly. How do you design it?

* Wrong answer: manually ACK-ing/NACK-ing UDP. More work for us, slower and fewer messages being sent.
* Data type: binary encoding, for maximum compression.
* Split data into smaller chunks, send in parallel. Each chunk should also indicate what number they are, for later verification.
* Use a mix of TCP and UDP. TCP announces the name of the data, the size of the data, and how many chunks will be sent. Receiver will prepare ports and create appropriate number of UDP sockets to receive the data. Send data in chunks via UDP, giving each chunk a number. Once all data sent, sender notifies the receiver. Receiver verifies whether chunks have all been received and communicate number of chunks and missed chunks to sender via TCP. Repeat until all data has been successfully received.

How do we get responses using UDP as it is stateless?

* Responders may send back a message after a timeout.

UDP messages can arrive out of order. When does that matter? If it does matter, how do you fix it? One example of when it doesn't matter.

* It matters when the data needs to be reconstructed in the correct order to be meaningful e.g. a very long text message or data structure.
* Assign each message some indicator of what position the message is among the overall data being sent, and let the receiver reconstruct it.
* An example of order not mattering is when messages sent via UDP aren't tightly related to each other to where ordering is required, e.g. emails being sent.

## Web Computing

What does "idempotent" mean?

* If you perform the same operation many times, you get the same result.

Is a Google search idempotent?

* It isn't. Performing a search changes the state in the server (tailor recommendation algorithm, for example).

What are some problems with PHP?

* Server is doing "display work", taking up resources.
* Hard to cache.
* Too much in one place - server code in client file.
* Can be insecure - code is running on your server.

Are parameterized paths better than GET queries?

* No. There are many valid ways to design APIs.

Can we have a single webserver that upholds REST seperation of responsibility?

* Webserver at www.myweb.com and API at api.myweb.com setup does not uphold location transparency, as they are located in different IPs.
* An `/api` path added to the website would be more transparent, while HTTP methods on the `/api` path can exclusively be reserved for API requests.

What are the pros and cons of server pages?

* Pros:
  * Improve client-side performance for more static sites by rendering content before it is sent to the client.
  * Better indexing for search engines and social media crawlers and accessibility by fully rendering the website.
* Cons:
  * The server bears the burden of rendering content over the client, increasing expenses.
  * Frequent server requests in more complex applications may actually decrease performance.

Would we pass information like username and password as our cookie? Why should or shouldn't we?

* No. Cookies are sent as part of the HTTP request, making it suspectible to packet sniffing, leading to sensitive information leakage.

What is a system that would keep you logged in without passing username in a cookie?

* Server sets cookie representing a session for the client. Client stays connected as long as the session has not timed out. Server needs to keep track of valid sessions.

What is the process of logging out?

* Client clears session ID from their cookies. Server must also renders the session invalid somehow.

## Server Performance

What does a web server do?

* Listen on a port
* Get new TCP client
* Get the request
* Fulfill that request

What if we get a lot of clients at once?

* We fulfill requests one at a time, leaving clients waiting

How can we serve more clients, faster?

* More servers -> more compute, but most expensive
* Better code -> expensive, time-consuming, sometimes marginal gains
* Do less work (caching) -> least expensive
* Multithreading

What is a thread?

* A thread is a stack and instruction pointer within a program. Multiple threads can be running different code concurrently, but they share the same memory.
* Note that Python threads (before Python 3.15) run concurrently but not in parallel, due to Python's GIL (Global Interpreter Lock) preventing more than one thread from running at a time. It's good for parallel I/O regardless

What is a cache?

* Key value pairs (usually). Store things which we can then fetch.

What is good to cache?

* Things that take time to generate - resource intensive. For example, suggestions (Netflix/Amazon) or trending things (items/movies).
* Most static responses can be cached. However, we can't cache if the operation isn't idempotent.

How do we handle DoS/DDoS?

* An error message to inform clients that we're temporarily down.
* Broadly, reconnect ASAP, restarting and reporting the error automatically. No one answer, but should not involve human interaction.

What are the best requests for DDoS-ing? Why?

* Lots of I/O, as they usually force waiting.
* Small request size, big response size.

What parts of a system can be DoS'd?

* CPU time
* Network
  * Bandwidth
  * Connections
* Memory

How do we address CPU problems?

* Make better use of the processor: don't write crummy code, leverage threading
* Throw more processing power at it: add more machines, or use better machines
* Break up the problem and make a new layer

How do we address bandwidth problems?

* Load balancer, multiple server locations
* Pass less data
* Rate limiting

How do we address connection problems?

* Shorter timeouts
* Horizontal scaling

How do we address memory problems?

* Write more efficient code
* Horizontal scaling
* Less caching

Is rate-limiting a solution to DoS?

* Kind of. It depends.

What are the HTTP headers related to rate-limiting?

* 429 Too Many Requests. Good users will respect a 429. Case study: Redit hug of death.

Is every outage due to (D)Dos?

* No. Example: AWS outage, which was due to DNS.

## Consistency

Why do we want consistency?

* A unique key should not be associated with more than one value.
* Everyone should have the same view of the world (within reason).

Is consistency inherently a distributed systems problem? Are all distributed systems subject to consistency/inconsistency?

* If all the state is in one place, no. Though, state isn't usually all in one place, so most of them are. Anytime state/data is replicated it's a possibility.

What could impact consistency?

* Network delays/unreliability (e.g. crashing, someone goes offline)
* Resource limitations (e.g. processor speed)
* Concurrency issues: poor programming, lack of coordination

Who is responsible for consistency?

* Whoever manages the state.

Do we need consistency?

* It depends. We may want to avoid it, because it's hard and potentially unnecessary.

How do we work around a lack of consistency?

* Avoid the need for consistency at all, by splitting up the state.
* Relax our consistency constraints, e.g. promising that we will eventually be consistent.

How do we detect inconsistency?

* Very hard. Mostly metadata and complex algorithms:
  * Version data
  * Timestamps, no perfect solution (e.g. malicious host or concurrent updates)
  * Operation logging
  * Checksums/comparing hashes
  * Monte Carlo technique, statistical evaluation
  * Randon spot checks


How do we resolve consistency?

* Many ways, depending on the application.
* Take the most recent
* Leave it up to the user (`git`)
* Go with the majority
* Have a "leader" dictating the state
* Two-phase commit is an imperfect, good way of preventing inconsistency, but doesn't fix it once it has happened.
