+++
date = "2013-05-31T22:33:44+02:00"
title = "SockJS for Go II (lessons learned)"

+++

It's been a while since implementing <a href="{{< relref "post/sockjs-for-go.md" >}}" target="_blank">SockJS library for Go</a>. Since then I learned couple of new things about the language and runtime. So here I'll try to summarise what it was.

<!--more-->
<h2>Ad: Concurrent Safe Data Structure of Connections </h2>
It's always a problem with shared data. I was choosing between using locks or channels and in the end I decided to go with channels (see <a href="https://github.com/igm/sockjs-go/blob/91373aa7f38d861719a8225a05abdd5f82904781/sockjs/sessions.go" target="_blank">sessions.go</a>). However this approach has some disadvantages. First of all it does not distinguish between read operations and write operations. And in fact what we can have is concurrent reads. We don't need to serialize read operations into channel. I've concluded that in this case it is better to use sync.RWLock. So I've changed the implementation to use sync.RLock()/RUnlock() for read operations and sync.Lock()/Unlock() for write operations (see <a href="https://github.com/igm/sockjs-go/blob/7f310b61fda162822d5006e0830dbe8d82d459ae/sockjs/sessions.go" target="_blank">sessions.go</a>).
<h2>Go Routines Leaking</h2>
It is easy to leak go routines. At first I did not realize that, but I started to worry after seeing memory consumption going up over time. After sending SIGQUIT signal (or Ctrl+\ on Linux) to the running process all was clear. Too many zombie routines hanging around. To demonstrate the problem have a look at this trivial example:

<script src="https://gist.github.com/igm/5687664.js"></script>
You can run it <a href="http://play.golang.org/p/MO8bN7_PqC" target="_blank">here</a>. In the output you'll see 2 routines. Notice routine 2 in "chan send" state. It is trying to send to the channel but that operation is never going to happen. And that is exactly what happened in my routines <a href="https://github.com/igm/sockjs-go/blob/b02995736d79745f95338a4a96e4e8dc9cbe47d2/sockjs/websocket.go#L58" target="_blank">here</a> and <a href="https://github.com/igm/sockjs-go/blob/cffe91abcddc3bd4c71350ec3b31fea08b381620/sockjs/rawwebsocket.go#L14" target="_blank">here</a>. One of the solution is to make channel "buffered". In this specific case it is enough to set buffer length to 1:  
<pre class="terminal">ch := make(chan bool, 1)</pre>
Other option would be to close the channel but that has different consequences (you can try it yourself and see what happens).
<h2>Conclusion</h2>
I have two sources to recommend that I consider very useful:
<ul>
<li> <a href="https://www.youtube.com/watch?v=QDDwwePbDtw" target="_blank">Google I/O 2013 - Advanced Go Concurrency Patterns by Sameer Ajmani </a>
</li>
<li> <a href="http://dave.cheney.net/2013/04/30/curious-channels" target="_blank">Curious Channels by Dave Cheney</a>
</li>
</ul>





