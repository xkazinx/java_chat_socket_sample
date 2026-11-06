# Java chat over sockets (2016)

A small chat written to learn plain TCP sockets in Java: a server that accepts any number of clients on one port and a Swing client that asks for a name and then shows the room's messages. No framework, no library beyond the JDK. A learning project from 2016, kept as it was.

## What it does

- The **server** listens on `127.0.0.1:9091`, accepts each connection and hands it to its own thread (`PeerHandler`). The handler asks the client for a name until it gets one nobody else uses, then relays every line the client sends to every connected client. Names and the writers are shared sets guarded with `synchronized`; a client that disconnects is removed from both.
- The **client** connects, answers the name prompt, and runs a reader thread that appends incoming lines to the window; the text field at the bottom sends a line per Enter. Leaving the window tells the room the user left.
- The **protocol** is three words over text lines: the server sends `SUBMITNAME`, the client answers with a name, the server replies `NAMEACCEPTED` and from then on every line is a chat message broadcast to all.

## What it shows

- `ServerSocket` / `Socket` with `BufferedReader` and `PrintWriter` over the streams, one thread per connection.
- A shared state (`HashSet<String>` of names, `HashSet<PrintWriter>` of outputs) touched only under a lock.
- A client split into transport (`Client`, with its reader thread), control (`Control`) and view (`GUI`, Swing with an auto-scrolling text area).

## Running it

Two IntelliJ projects, `server/` and `client/`, each with a `Main`.

1. Run `server/src/com/company/Main.java`: it prints `Socket opened at 127.0.0.1:9091`.
2. Run `client/src/com/company/Main.java` as many times as you want users; each window asks for a name and joins the room.

The address and port are constants in `server/.../Main.java` and `client/.../Main.java`; change both to chat across machines.

## Layout

```
server/src/com/company/Main.java           names and writers of the room, starts the listener
server/src/com/company/SocketHandler.java  ServerSocket, accept loop, one PeerHandler per client
server/src/com/company/PeerHandler.java    the name handshake, then relaying lines to everyone
client/src/com/company/Main.java           starts the client against the server
client/src/com/company/Client.java         the socket, its streams and the reader thread
client/src/com/company/Control.java        wires the window to the client
client/src/Visual/GUI.java                 the Swing window
```
