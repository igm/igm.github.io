+++
date = "2014-12-06T22:31:13+02:00"
title = "SockJS for Go III (yet another take)"
+++

It's been a while since the last blog and this is the last in the series of "SockJS-go lessons learned". 

<!--more-->

## Version "v2"

"v2" is a compete rewrite of the SockJS-go library [1], the entire code base is much smaller thanks to using the full potential of Go's <a href="http://godoc.org/net/http" target="_blank">net/http</a> library. So no more http "hijacks" and number of go routines the library spawns internally is kept to minimum. The project also adopted versioning based on <a href="http://labix.org/gopkg.in" target="_blank">gopkg.in</a>. The main reason for gopkg.in adoption was that "v2" uses a different API compared to "v1", which was for experimental purposes only. The aim of "v2" is to become more stable and eventually production ready library. To get the package execute:

	$ go get gopkg.in/igm/sockjs-go.v2/sockjs

To import this package, add the following line to your code:

	import "gopkg.in/igm/sockjs-go.v2/sockjs"

## Data frames
SockJS was primarily created to enable applications running in older browsers to use WebSocket-like communication with server, or in other words provide <a href="https://github.com/sockjs/sockjs-node" target="_blank">WebSocket emulation</a>&nbsp;[2]. It provides a mechanism to send messages (or data frames) in both directions, from client to server and server to client. The only properly supported frames in SockJS are text frames. So in order to send binary data it is solely responsibility of an application to properly encode/decode binary message into text messages using UTF-8 encoding. Just to make the entire picture more complete, WebSockets as defined by RFC6455 support both types of data frames: <a href="http://tools.ietf.org/html/rfc6455#section-5.6" target="_blank">binary and text</a>.

## API Changes
As described above, SockJS (similar to WebSocket) uses data frames. A client initiates a connection to a server and creates a new sockjs session. Depending on the underlaying transport, the session can span over multiple connections. Thus the notion of sockjs session. This is reflected in the API by <a href="http://godoc.org/github.com/igm/sockjs-go/sockjs#Session" target="_blank">sockjs.Session</a> interface. Interface is simple enough to enable receiving and sending text frames, get the session ID and close the session.

## SockJS Protocol Tests
This is probably the most questioned part of SockJS for Go implementation. Not all tests pass, and there's no plan to make them pass. However here is the list of tests that do not pass with some explanation:

* <b>WebSocket tests</b> protocol tests use hixie-76 and hybi-10 draft implementations of WebSocket protocol. The latest RFC6544 is not covered. There are no plans to implement draft versions in Go (SockJS-go uses <a href="http://www.gorillatoolkit.org/pkg/websocket" target="_blank">Gorilla's WebSockets</a> implementation). This also means, that older browsers implementing draft versions will not use WebSocket but will fall back to another transport protocol
* <b>JSONEncoding tests</b> use escaping of UTF surrogates to enable binary frames. As Go uses UTF-8 encoding internally for strings, it's advisable to use only valid characters in frames. Any binary encoding/decoding is responsibility of application layer and SockJS-go does not do any escaping.
* <b>RawWebsocket tests</b> test raw WebSockets. SockJS-go test server registers handlers for raw WebSocket even if they don't pass (due to above mentioned WebSocket draft protocol reasons). If in your application you plan to use raw WebSockets you can, the net/http package is flexible enough to mix and match various handlers so you can use SockJS-go handler together with any other http handler including WebSocket handlers like <a href="http://godoc.org/code.google.com/p/go.net/websocket" target="_blank">this</a> or <a href="http://www.gorillatoolkit.org/pkg/websocket" target="_blank">this</a>.

## Conclusion
Rewriting the SockJS-go library several times (9 or 10 times all together) was a perfect drill to learn Go more in deep. The latest version tries to be as simple as possible. It is also incredible to see the progress of Go since it was first introduced. The language is very much the same, but the libraries and runtime are improving at a fast pace with every release keeping the backward compatibility.

## References
1. <a href="https://github.com/igm/sockjs-go" target="_blank">SockJS-go GitHub repository</a></li>
2. <a href="https://github.com/sockjs/sockjs-node">SockJS-node</a>&nbsp;</li>

