## URI Schemes
### `urn:uuid:<lower case hex with hyphens>`
See [RFC 4122](https://www.rfc-editor.org/rfc/rfc4122.html).

### `command:<path>`
Designates the _type_ of command, e.g. `command:rooms/join`.

### `error:<path>`
Designates the _type_ of error, e.g. `error:messages/oversized`.

### `event:<path>`
Designates the _type_ of event, e.g. `event:rooms/messages`.

### `query:<path>`
Designates the _type_ of query, e.g. `query:rooms/messages`

### `answer:<path>`
Designates the type of _query_ for which this is an answer, i.e.
each `<path>` corresponds to a `query:<path>`.

### `message:<uuid>`
Designates a particular message, e.g.
`message:urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6`.

### `user:<uuid>`
Designates a particular user, e.g.
`user:urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6`.

### `room:<uuid>`
Designates a particular room, e.g.
`room:urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6`.

### `profile:<path>`
Designates a kind of field in a profile, e.g. `profile:display-name`.

## Request/Response/Event Wrapper
```js
{
    "ID": URI("urn:uuid"),
    "response to?": URI("urn:uuid")
    "payload": Object
}
```

## Request/Response/Event Payloads
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
    "server time": Date,
    "rooms": [URI("room"), ...etc]
}}
```
```js
{"error:rooms/unauthorized": {
    "rooms": {
        [URI("room")]: {
            "current level": or(Number, null),
            "required level": or(Number, null)
        },
        ...etc
    }
    "diagnostic": String
}}
```
```js
{"error:rooms/invalid": {
    "rooms": [URI("room"), ...etc],
    "diagnostic": String
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
    "server time": Date,
    "parent room": URI("room"),
    "room": URI("room")
}}
```
```js
Schema("error:rooms/unauthorized")
```
```js
Schema("error:rooms/invalid")
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
{"error:rooms/nonmember": {
    "room": URI("room"),
    "user": URI("user")
}}
```
```js
Schema("error:rooms/invalid")
```

`command:rooms/relevel`
----------------------
### Request
```js
{"command:rooms/relevel": {
    "room": URI("room")
    "user": URI("user"),
    "old level": or(Number, null),
    "new level": or(Number, null),
    "message?": String
}}
```
### Response
```js
{"event:rooms/relevel": {
    "server time": Date,
    "user": URI("user"),
    "old level": or(Number, null),
    "new level": or(Number, null),
    "message?": String
}}
```
```js
{"error:rooms/stale-level": {
    "user": URI("user")
    "stated level": or(Number, null),
    "actual level": or(Number, null)
}}
```

`command:rooms/subscription`
-------------------------
### Request
```js
{"command:rooms/subscription": {
    "rooms": {
        [URI("room")]: {
            "last message?": URI("message")
        },
        ...etc
    }
}}
```

### Response
```js
{"event:rooms/subscription": {
    "server time": Date,
    "rooms": [URI("room"), ...etc],
    "errors": {
        [URI("room")]: or(
            Schema("error:/rooms/unauthorized"),
            Schema("error:/rooms/invalid"),
            Schema("error:/messages/invalid")),
        ...etc
    }
}}
```

`command:users/fork`
--------------------
### Request
```js
{"command:users/fork": {
    "display name?": String
}}
```
### Response
```js
{"event:users/fork": {
    "server time": Date,
    "user": URI("user"),
    "reset token": URI("urn:uuid")
}}
```

`command:users/profile`
-----------------------
### Request
```js
{"command:users/profile": {
    "profile": [Schema("profile"), ...etc]
}}
```
### Response
```js
{"event:/users/profile": {
    "server time": Date,
    "user": URI("user"),
    "profile": [Schema("profile"), ...etc]
}}
```
```js
{"error:/users/profile/invalid": {
    "user": URI("user"),
    "errors": {
        [URI("profile")]: {
            "diagnostic": String
        }
    }
}}
```
```js
{"error:/users/profile/throttled": {
    "user": URI("user"),
    "actual per second": Number,
    "maximum per second": Number
}}
```

`query:users/profile`
---------------------
### Request
```js
{"query:users/profile": {
    "profile": [URI("profile"), ...etc]
    "users": [URI("user"), ...etc] 
}};
```
### Response
```js
{"answer:users/profile": {
    "profiles": {
        [URI("user")]: [Schema("profile"), ...etc]
    },
    "errors": {
        [URI("user")]: [
            or(Schema("error:users/invalid"),
               Schema("error:profiles/invalid")),
            ...etc
        ]
    }
}}
```

`command:rooms/post`
--------------------
### Request
```js
{"command:rooms/post": {
    "room": URI("room"),
    "text": String,
    "MIME?": String  // default: text/markdown
}}
```
### Response
```js
{"event:rooms/post": {
    "server time": Date,
    "room": URI("room"),
    "message": URI("message")
}}
```
```js
Schema("error:rooms/invalid")
```
```js
Schema("error:rooms/unauthorized")
```
```js
{"error:messages/oversized": {
    "room": URI("room"),
    "actual bytes": Number,
    "maximum bytes": Number
}}
```
```js
{"error:rooms/throttled": {
    "room": URI("room"),
    "actual per second": Number,
    "maximum per second": Number
}}
```

`event:rooms/message`
---------------------
```js
{"event:rooms/messages": {
    "messages": {
        [URI("message")]: {
            "server time": Date,
            "room": URI("room"),
            "user": URI("user"),
            "text": String,
            "MIME": String
        }
    }
}}
```

`query:rooms/participants`
--------------------------
### Request
```js
{"query:rooms/participants": {
    "rooms": [URI("room"), ...etc]
}}
```
### Response
```js
{"answer:rooms/participants": {
    "rooms": {
        [URI("room")]: [{
            [URI("user")]: {
                "level": Number
            }
        }]
    },
    "errors": {
        [URI("room")]: or(
            Schema("error:/rooms/unauthorized"),
            Schema("error:/rooms/invalid")),
        ...etc
    }
}}
```

`query:rooms/level`
-------------------
### Request
```js
{"query:rooms/level": {
    "rooms?": [URI("room"), ...etc]
}}
```
### Response
```js
{"answer:rooms/level": {
    "rooms": {
        [URI("room")]: {
            "level": Number
        }
    },
    "errors": {
        [URI("room")]: Schema("error:rooms/invalid")
    }
}}
```

## Profile Fields
```js
{"profile:display-name": String}
```
