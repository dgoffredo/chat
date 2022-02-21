Chat
====
A design for a chatroom system.  Working on it, anyway.

Goals
-----
- simple
- secure enough to actually use on the internet
- easy to host on one node
- possible to host on many nodes
- thousands of connections per node
- complete server
- minimal client
- no XML

So Far
------
Check out [concepts][4] for an overview of what's peculiar about this design.

Clients enter the system using a few HTTP [endpoints][1], and
afterwards switch to an application specific [protocol][2] over
[WebSockets][3].

The the server's data model is described in [model.md](model.md).

[1]: endpoints.md
[2]: protocol.md
[3]: https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
[4]: concepts.md
