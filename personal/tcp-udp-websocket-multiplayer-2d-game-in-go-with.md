---
tags:
  - links
  - bookmarks
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# TCP/UDP/WebSocket multiplayer 2D game in Go (with web client), for fun - golang

Created: 2019年7月18日 上午8:47
URL: https://www.reddit.com/r/golang/comments/3gt6vm/tcpudpwebsocket_multiplayer_2d_game_in_go_with/

![mFIzFfx6BCIkqdRGS0ihN4BbT0JE4866xDW22gUc1A4.jpg](TCP%20UDP%20WebSocket%20multiplayer%202D%20game%20in%20Go%20(with%20/mFIzFfx6BCIkqdRGS0ihN4BbT0JE4866xDW22gUc1A4.jpg)

(This is a cross post from [r/gogamedev](https://www.reddit.com/r/gogamedev/), but I thought it'd be interesting for people here since this is pure Go and it deals with interesting things like `net.Conn`, channels/select statements, etc., and having it work cross-platform.)

I'm a big fan of Go and I think it has amazing potential for making games. I'm interested in pushing its limits and finding out how far it can go, and so far it's a blast!

I wanted to share a little game project I'm working on in my spare time. It's written 100% in pure Go, aside from some cgo dependencies for OpenGL and GLFW. It's actually a port of an unfinished original version I started many years ago in C++.

I took a look at Go's `net` package and wanted to see how hard it'd be to send some TCP/UDP packets and have it connect to the C++ server... then ported the server, added logic, a simple renderer, and by now a large part is working in Go.

Here's a [screenshot](https://raw.githubusercontent.com/shurcooL/eX0/master/eX0-go/Screenshot.png) to give you an idea. I'm working on the netcode/gameplay first, so the graphics are very basic for now. The C++ version looked [slightly better](https://camo.githubusercontent.com/89b4b80e3237e59f703ec814a10b6f9c55922ebf/68747470733a2f2f646c2e64726f70626f7875736572636f6e74656e742e636f6d2f752f383535343234322f646d697472692f70726f6a656374732f6558302f53637265656e73686f742e706e67).

Anyway, let me get to the cool part. One of the goals I had when I started the Go port was to be able to have the game client run in a browser. I wanted to use GopherJS compiler and WebGL.

But there's no easy way to send UDP packets form the browser, so how did I get around that? For now, I use a WebSocket connection (which is a TCP-like connection) and marshall my TCP+UDP packets over that stream. It was great to be able to do this in Go because I could use channels/select statements (that I never had in C++) like you can see [here](https://github.com/shurcooL/eX0/blob/91ae2559d3e860ff7454f330ff6efe2b92b9110e/eX0-go/net_tcp.go#L22-L103).

The original networking protocol uses TCP/UDP packets (and my Go version is currently staying compatible with the original C++ version), but I have 3 transport types: normal UDP/TCP via "net" package (see `net.go` file). Combined virtual TCP+UDP mode over WebSocket (`net_tcp.go`). And using Go channels (`net_chan.go`), which obviously only works when both the client and server are running in the same process. It gets unbeatably low ping times though!

I said there's a web client, and you can try that just by clicking on this link in any modern browser (it even works on mobile, but I don't have touch controls):

Arrow keys and W/A/S/D to move around. That's it for now.

(Try opening two windows if there's no one else online, and try moving around. Open dev console to see networking info.)

The source code is all here at [https://github.com/shurcooL/eX0/tree/master/eX0-go](https://github.com/shurcooL/eX0/tree/master/eX0-go#ex0-go-). Feel free to star the repo or watch it for further development news. I'm happy to answer questions (with possible delay)!