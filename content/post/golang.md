+++
date = "2012-09-06"
draft = false
title = "go get..."

+++

Recently I came across a programming language called <a href="http://golang.org/" target="_blank">GO</a>. At first I was a little skeptical about it, why another language? I spent most of my IT life in Java world, occasionally touching other languages (Ruby, C++, JavaScript, Python, Groovy, Scala, Erlang ...) but GO really got my attention. 
<!--more-->

GO is open source programming language, it is productive, concurrent, garbage-collected and builds fast (really fast) due to simple and effective dependency model. Without going into details about the main features of the language (<a href="http://tour.golang.org/" target="_blank">A Tour of Go</a> is the best place to start) here I want to demonstrate the simplicity of installation and usage.

## Installation
Installation process is simple. <a href="http://golang.org/doc/install" target="_blank">Download</a> the version specific for your operating system and install/unzip. Then two environment variables need to be set in order to integrate GO into system: GOROOT and GOPATH (and optionally PATH).

	$ export GOROOT=/usr/local/go
	$ export GOPATH=~/gobase
	$ export PATH=$GOROOT/bin:$PATH
	$ go env
	GOROOT="/usr/local/go"
	GOBIN=""
	GOARCH="amd64"
	GOCHAR="6"
	GOOS="darwin"
	GOEXE=""
	GOHOSTARCH="amd64"
	GOHOSTOS="darwin"
	GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
	GOGCCFLAGS="-g -O2 -fPIC -m64 -pthread -fno-common"
	CGO_ENABLED="1"

Then we need to create a directory structure according to convention:

	$ mkdir -P $GOPATH/bin
	$ mkdir -P $GOPATH/pkg
	$ mkdir -P $GOPATH/src

Another not really needed, but useful step is to add $GOPATH/bin to your PATH variable:

	export PATH=$GOPATH/bin:$PATH

## Test Drive
Let's test our configuration by installing "Hello World" sample I've created and put on <a href="https://github.com/igm/go-sample/blob/master/hello/hello.go" target="_blank">github</a>.

	$ go get github.com/igm/go-sample/hello
	$ $GOPATH/bin/hello
	Hello World!

## What just happened?
GO downloaded my package called "hello" from github repository, fetched any external dependencies (source code), compiled it and placed resulting hello binary into GOPATH/bin directory. No need to fetch the source manually, create complex build files (Makefile, Ant, Gradle, Sbt,...), invoke build process and hope all goes well. It all just happend in one command. Lets try another example, simple HTTP server:

	$ go get github.com/igm/go-sample/webserver
	$ $GOPATH/bin/webserver

Now open the browser and navigate to <a href="http://localhost:8080/" target="_blank">http://localhost:8080</a>. I want to emphasize that both binary files (hello and webserver) are now GO independent and self-contained applications which basically means you can share them with your friend or customer assuming they are on the same OS architecture as you are (or you can create a binary for their specific OS architecture). They don't need to install anything in order to run them, it just works.

## Summary
I think GO will become more popular in the following months. It is ideal language for server side programming and I'm sure we'll see it more integrated into the most popular Linux distributions. And of course, you can deploy your GO application to <a href="https://developers.google.com/appengine/docs/go/" target="_blank">App Engine</a>.

## References
1. <a href="http://golang.org/" target="_blank">GO Home Page</a>
2. <a href="http://golang.org/doc/install" target="_blank">GO Installation instructions</a>
3. <a _blank="_blank" href="http://tour.golang.org/" target="_blank">A Tour of Go</a>
4. <a _blank="_blank" href="https://developers.google.com/appengine/docs/go/" target="_blank">App Engine GO Runtime</a>
5. <a href="https://gist.github.com/3731476" target="_blank">The Case for #golang</a>


