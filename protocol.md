## URI Schemas
### `urn:uuid:<lower case hex with hyphens>`
See [RFC 4122](https://www.rfc-editor.org/rfc/rfc4122.html).

### `command:<path>`
### `error:<path>`
### `event:<path>`
### `query:<path>`
### `message:<path>`
### `user:<uuid>`
### `room:<uuid>`

## Message Wrapper
```js
{
    "ID": URI("urn:uuid"),
    "response to?": URI("urn:uuid")
    "payload": MessagePayload
}
```

## Message Payloads
`command:rooms/join`
--------------------
### Request
```js
{"command:rooms/join": {
    "rooms": [URI("room"), ...etc] 
}}
```
### Response
```js
{"event:rooms/join": {
    "rooms": [URI("room"), ...etc]
}}
```
```js
{"error:rooms/unauthorized": {
    "rooms": {
        [URI("room")]: {
            "current level": or(Number, null),
            "required level": or(Number, null)
        }
    }
    "message": String
}}
```
```js
{"error:rooms/invalid": {
    "rooms": [URI("room"), ...etc],
    "message": String
}}
```

`command:rooms/fork`
--------------------
### Request
```js
{"command:rooms/fork": {
    "room": URI("room"),
    "founder level?": Integer  // default: 4, constraint: >= 0
}}
```
### Response
```js
{"event:rooms/fork": {
    "room": URI("room")
}}
```
```js
{"error:rooms/nonmember": {
    "room": URI("room"),
    "user": URI("user")
}}
```
```js
"error:rooms/invalid"
```

`command:rooms/leave`
---------------------
### Request
```js
{"command:rooms/leave": {
    "rooms": [URI("room"), ...etc]
}}
```
### Response
```js
"error:rooms/nonmember"
```
```js
"error:rooms/invalid"
```

`command:rooms/relevel`
----------------------
### Request
```js
{"command:rooms/relevel": {
    "room": URI("room")
    "user": URI("user"),
    "old level": or(Number, null),
    "new level": or(Number, null)
}}
```
### Response
```js
{"event:rooms/relevel": {
    "user": URI("user"),
    "old level": or(Number, null),
    "new level": or(Number, null)
}}
```
```js
{"error:rooms/stale-level": {
    "user": URI("user")
    "stated level": or(Number, null),
    "actual level": or(Number, null)
}}
```
```js
"error:rooms/nonmember"
```
```js
"error:rooms/invalid"
```
```js
"error:rooms/unauthorized"
```
`command:rooms/subscribe`
-------------------------
### Request
```js
{"command:rooms/subscription": {
    "subscribe": or("all", [URI("room", ...etc)]),
    "unsubscribe": or("all", [URI("room", ...etc)])
}}
```
### Response
TODO

`command:users/fork`
--------------------
### Request
```js
{"command:users/fork": {}}
```
### Response
```js
{"event:users/fork": {
    "user": URI("user"),
    "login token": URI("urn:uuid")
}}
```
TODO

TODO
----
TODO
