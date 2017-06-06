+++
date = "2012-12-11T22:37:00+02:00"
title = "SockJS for Go"

+++
<a href="https://github.com/sockjs/sockjs-client" target="_blank">SockJS</a> is a browser JavaScript library that provides a WebSocket-like object. SockJS gives you a coherent, cross-browser, Javascript API which creates a low latency, full duplex, cross-domain communication channel between the browser and the web server. Under the hood SockJS tries to use native WebSockets first. If that fails it can use a variety of browser-specific transport protocols and presents them through WebSocket-like abstractions. 

<!--more-->

SockJS is intended to work for all modern browsers and in environments which don't support WebSocket protcol, for example behind restrictive corporate proxies. What I was missing was a server side support library for GO programing language. So I decided to <a href="https://github.com/igm/sockjs-go" target="_blank">create one myself</a>.  The protocol itself is not difficult to understand. It tries to simulate websocket functionality. From the JavaScript client the server needs to be able to receive frames represented in JSON format. 

	"hello world"
	{"name":"john","surname":"doe"}

According to the <a href="https://github.com/sockjs/sockjs-protocol/issues/28" target="_blank">protocol specification</a> these messages are sent in an array:

	["hello world"]
	[{"name":"john","surname":"doe"}]

And on the other side server needs to be able to send back control frames (heartbeat, open frame, close frame) and data frames, also in JSON format. The supported transport protocols for sending messages to the client can be divided into two groups:

* <b>Streaming</b>: XHR streaming, EventSource, Websocket
* <b>Non streaming</b>: XHR polling, JSONP

In terms of receiving messages, there are 3 methods: XHR, JSONP and Websocket.
On top of that IFrame and HtmlFile technique is used to overcome various browser specific limitations in order to support transport protocols mentioned above. The SockJS project comes with a <a href="https://github.com/sockjs/sockjs-protocol" target="_blank">convenient set of protocol tests</a> that guarantee the correctness of server side implementation to a certain degree.

## Channels and Locks
All of the protocols define a certain state of communication. <b>Open</b> state when a connection is established for the first time, then there's a <b>passive</b> state representing existing connection but client is currently disconnected (non streaming protocols) and then the <b>active</b> state when a connection between client and server exists and is active. The last state is connection <b>closed</b> meaning client or server closed the connection and after a certain period of time the connection expires completely and is deleted which is a terminal state. I will not go much into details of protocol specification, rather I'll focus on two problems I faced and how Go language features helped to solve them.

## Parallel Requests from Client
At any time if a connection is in active state, all other connection requests should be rejected with a close message 
<pre class="terminal">[2010,"Another connection still open"].</pre>
The idiomatic solution in other programming languages is to use a flag associated with a connection and each time a connection gets into active state set the flag and reject all other connections. Using Go I opted for a different approach - <b>Go routines and channels</b>, and it turned out to be an efficient one. The main inspiration came from Rob Pike's presentation <a href="https://www.youtube.com/watch?v=HxaD_trXwRE" targe="_blank">Lexical Scanning in Go - Rob Pike</a>. The presentation shows how to implement a state machine using function types, so I defined a function type as follows:

	type connectionStateFn func(*conn) connectionStateFn

State is a function, which using a connection as an input parameter performs required logic on connection object (reads HTTP transaction request/response objects) and returns back another state function for further connection processing until it finishes. And the actual execution happens in a separate Go routine.

<script src="https://gist.github.com/4253786.js?file=state.go"></script>Now the question is, how does state function process HTTP request/response? The answer is simple and goes hand in hand with Go routines: via <b>channel</b>. We can look at a connection as an object that is alive. It's not just data manipulated by other "threads" as in other programming languages (in Java for example it's very common to process requests in one of the threads from thread pool and access shared data from that thread). Here a communication is a "thread" itself. I put "thread" in parenthesis on purpose, as it's not a real thread but rather a lightweight Go routine. 

Then all the requests that come to the http handler are put into a connection specific input channel and processed sequentially in a concurrent safe way. The state function for open connection looks for example like this:
<script src="https://gist.github.com/4253741.js"></script> Let's look at "already active connection" implementation:
<script src="https://gist.github.com/4253724.js?file=active.go"></script>Entire code for all the states can be found <a href="https://github.com/igm/sockjs-go/blob/master/sockjs/handler.go" target="_blank">here</a>. No locks, the code is simple and easy to understand which I think is mostly thanks to not having to bother with locks.

## Concurrent Safe Data Structure of Connections
Another problem that arose was related to a "global" data structure holding all the connections associated to session id in a map. Data structures in Go are not concurrent safe by default and there's a reason for that. They are just simple and effective low level components to be used to built up more complex structures and they serve that purpose very well. The word "global" is very important here. Any data not private to a go routine and accessible from several go routines need to be protected. Again, there are two options here: <b>locks</b> or <b>channels</b>. Let's look at both approaches and compare them.
### Locks
The concurrent safe code using locks looks like this:
<script src="https://gist.github.com/4253873.js?file=locks.go"></script>Each time there's an access to the map a mutex is locked and in the end unlocked. This is an idiomatic approach you'd use in most programming languages.
### Channels
<i>"Don't communicate by sharing memory; share memory by communicating."</i> Let's look at the implementation using channels. Instead of using Lock, we'll create a channel of functions that will be executed in a Go routine specific to the structure:
<script src="https://gist.github.com/4253953.js"></script>The map is more than a structure now. The map is a routine that "lives" its own life. Instead of accessing map directly, we send actions via channel and these actions are executed sequentially and thus the implementation is also concurrent safe. In case of get operation we are interested in result, that's why there's a "ret" notification channel. In case of set operation we don't need to wait (as there's no return value) and actual set operation is non-blocking. In some cases this can be an advantage not to be blocked. And inconsistency can't happen. If we invoke get operation right after set operation, the get will be queued in channel and will be executed only after the set operation is finished.

Here I'm not trying to persuade to use channels instead of locks. Instead on these simple examples with map I want to describe an approach using a channel. Instead of looking at the structure as a data, we can look at it as a micro system (with its own life) that accepts inputs through channel. In case of more complex structures and operations that can be performed on them the implementation might turn out to be easier to understand and implement compared to lock-based approach.

## References
1. <a href="http://golangtutorials.blogspot.ie/2011/05/table-of-contents.html" target="_blank">GoLang Tutorials</a>
2. <a href="http://golang.org/" target="_blank">The Go Programming Language</a>
3. <a href="https://www.youtube.com/watch?v=HxaD_trXwRE" targe="_blank">Lexical Scanning in Go - Rob Pike</a>
