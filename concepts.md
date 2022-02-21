Concepts
========
There are a few things about the design of this chatroom system
that might be unusual.  This document is an overview of the design,
with an emphasis on the unusual parts.

Domain
------
_Users_ send _messages_ to _rooms_.  That much you could have guessed.

Levels
------
Instead of having _administrators_ who can _ban_ users, each participant in a
room has associated with them an integer _level_.  Users who have never been
invited to the room have a `null` level.

A level is specific to a `(room, user)` pair, i.e. a user can have different
levels in different rooms.

Any user with a non-`null`, nonnegative level may set another user's `null` level to `0`.
This is called an invitation.

Any user with a nonnegative level `L` may promote another user from at least
`-L` to at most `L`. 

Any user with a nonnegative level `L` may demote another user from at most
`L - 1` to at least `-L`.

Note that this means that power is viral.  Once you promote a user to a level
`L`, they may promote anybody to that level indiscriminantly.  On the other
hand, if your level is higher than theirs, then you can demote them and anybody
they promoted.

Negative levels are bans.  A user whose level is less than zero cannot send
messages to the room or read messages from the room.  Banned users cannot
invite, promote, or demote any users in the room.  There are multiple levels of
ban corresponding to the "maximum banning potential" of the user who issued
the ban.  A level 3 user can issue a level -1, -2, or -3 ban, while a level 4
user can issue those bans in addition to a level -4 ban.

Forking Rooms
-------------
Like processes in Unix, a room is never created from nothing.  Instead, a room
is _forked_ from another room known as its _parent room_.

A forked room has its own ID, but shares everything else with its parent.

- A forked room's participants begin as whichever participants were in the
  parent room when the fork occurred, including participant levels (possibly
  with the exception of the participant performing the fork).
- A forked room's message history begins as whatever the message history of
  the parent room was when the fork occurred.

To fork a room, the performing user must have a non-`null`, non-negative level
in the room.  When the user performs the fork, they may specify an arbitrary
non-negative level for themselves in the new room (the "founder level").  The
founder level defaults to 4, allowing for some power hierarchy.

When a room is forked, an invitation is sent to each of its participants.  This
makes the "fork/execv" analogy impossible, since invitations might be sent
before you're able to uninvite existing participants.  Well, possible but rude.

Each server may have its own conventions about which room should be the
default for forking, e.g. "room:urn:uuid:00000000-0000-0000-0000-000000000000".

Direct Messages
---------------
There are no direct messages.  Instead, you fork a room containing only
yourself, then invite the other user, and then send messages to that room.

Forking Users
-------------
Users, like rooms, cannot be created from nothing.  Instead, a user is _forked_
from another user known as their _parent user_.

A forked room has its own ID and its own credentials, but shares everything
else with its parent.

- A forked user is a participant in each room in which the parent user was a
  participant when the fork occurred, including the levels.
- A forked user's message history is the same as the parent's when the fork
  occurred.

When a user is forked, the new user has an associated auth reset token, which
the new user can use to set a new authentication password.  The basic auth
username is the associated user ID, e.g.
"user:urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6".

Each server may have its own conventions about which user should be the
default for forking, e.g. "user:urn:uuid:00000000-0000-0000-0000-000000000000".
