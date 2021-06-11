---
layout: post title:  How to write the simplest chat server in golang categories: [GoLang]
---

### Introduction

In this tutorial, we implement a very simple chat server with golang that uses golang standard libraries. All we need is
a broadcaster method that sends messages to clients and a handler method for handling the connection.

So first of all, we need a connection and we can use `net` package:

``` go
func main() {
   listen, err := net.Listen("tcp", "localhost:8080")
   if err != nil {
      return
   }
}
```

After that for accepting connections we need an infinite loop :

``` go
for {
   conn, err := listen.Accept()
   if err != nil {
      log.Print(err)
      continue
   }
}
```

### Handler

Now we must handle a connection on the chat server. We create a map of client address and connection and broadcast a
message that notices every one a client joined after that we must broadcast every single message of the client to other
clients at the end we broadcast a message to everyone if any client closes its connection.

So we need a map, two channels, and a message struct:

``` go
var clients = make(map[string]net.Conn)
var leaving = make(chan message)
var messages = make(chan message)

type message struct {
   text    string
   address string
}
func handle(conn net.Conn) {
   clients[conn.RemoteAddr().String()] = conn

   messages <- newMessage(" joined.", conn)

   input := bufio.NewScanner(conn)
   for input.Scan() {
      messages <- newMessage(": "+input.Text(), conn)
   }
   //Delete client form map
   delete(clients, conn.RemoteAddr().String())

   leaving <- newMessage(" has left.", conn)

   conn.Close() // ignore errors
}
func newMessage(msg string, conn net.Conn) message {
   addr := conn.RemoteAddr().String()
   return message{
      text:    addr + msg,
      address: addr,
   }
}
```

### Broadcaster

So we need a broadcaster to broadcast all messages to all clients we can use select we heave two cases one for all
messages and the other one is for leaving messages they listen to channels iterate on our client map skip the message
producer and write a message on connection

```go
func broadcaster() {
   for {
      select {
      case msg := <-messages:
         for _, conn := range clients {
            if msg.address == conn.RemoteAddr().String() {
               continue
            }
            fmt.Fprintln(conn, msg.text) // NOTE: ignoring network errors
         }

      case msg := <-leaving:
         for _, conn := range clients {
            fmt.Fprintln(conn, msg.text) // NOTE: ignoring network errors
         }

      }
   }
}
```

The leaving message doesn’t need to skip the message owner because it’s removed from the map already. Now let’s complete
the main method:

``` go
func main() {
   listen, err := net.Listen("tcp", "localhost:8080")
   if err != nil {
      log.Fatal(err)
   }

   go broadcaster()
   for {
      conn, err := listen.Accept()
      if err != nil {
         log.Print(err)
         continue
      }
      go handle(conn)
   }
}
```

As you see we have tree goroutines here one of them in the main goroutine that listen for and accept incoming network
connections from clients.The broadcaster goroutine there is only one of it in the application lifetime, but we have to
handle goroutine per client. While hosting a chat session for n clients, this program runs 1n+2 concurrently
communicating goroutines. Now let’s test it

``` go
package main

import (
   "io"
   "log"
   "net"
   "os"
)

func main() {
   conn, err := net.Dial("tcp", "localhost:8080")
   if err != nil {
      log.Fatal(err)
   }
   done := make(chan struct{})
   go func() {
      io.Copy(os.Stdout, conn) // NOTE: ignoring errors
      log.Println("done")
      done <- struct{}{} // signal the main goroutine
   }()
   mustCopy(conn, os.Stdin)
   conn.Close()
   <-done // wait for background goroutine to finish
}

func mustCopy(dst io.Writer, src io.Reader) {
   if _, err := io.Copy(dst, src); err != nil {
      log.Fatal(err)
   }
}
```
This is our client app its simply send message from stdin to server and write messages to stdout thats it.

![Chat server test](https://miro.medium.com/max/1280/1*KsEM3ZLwCYGg__M_xkuhVg.gif)

I hope you enjoy it ;)

You can find the source code [here](https://github.com/mehranus/chat-server)

Fork and star it if you enjoy

You can also read this article on Medium [here](https://mehranbehnam77.medium.com/how-to-write-the-simplest-chat-server-in-golang-f70ba7abd94a)