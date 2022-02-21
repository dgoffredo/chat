HTTPS Endpoints
===============
`/api`
------
WebSockets protocol upgrade, after which the connected client
speaks with the server using the application [protocol](protocol.md).
Access will be denied unless the request has a cookie set containing a
valid login token.

`/auth/login`
-------------
Use [basic access authentication][1] in the request, and if successful,
the server will set a cookie containing a login token.

`/auth/reset`
-------------
Specify a valid reset token in the request, together with [basic access
authentication][1], and if successful, the server will overwrite existing
credentials with those specified in the basic auth.

[1]: https://en.wikipedia.org/wiki/Basic_access_authentication
