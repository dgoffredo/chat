HTTPS Endpoints
===============
`GET /api`
------
WebSockets protocol upgrade, after which the connected client
speaks with the server using the application [protocol](protocol.md).
Access will be denied unless the request has a cookie set containing a
valid login token.

`GET /auth/login`
-------------
Use [basic access authentication][1] in the request, and if successful,
the server will set a cookie containing a login token.  A JSON message
containing the token will also be returned as the response body:
```js
{
    "user": URI("user"),
    "login token": URI("urn:uuid"),
    "expiry": Date
}
```

`POST /auth/reset`
-------------
Specify a valid reset token in the `X-Password-Reset-Token` request header,
together with [basic access authentication][1], and if successful, the server
will overwrite the existing password (if any) with that specified in the basic
auth.  The basic auth username is the associated user ID, e.g.
"user:urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6".

[1]: https://en.wikipedia.org/wiki/Basic_access_authentication
