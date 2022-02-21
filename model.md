Data Model
==========
At the bottom of it all are relational database tables, but there is a larger
structure.

Transition Ledger, Volatile State, and Message History
------------------------------------------------------
Those are the three categories.

### Transition Ledger
I was going to call this "event ledger," but since I excluded messages it
seemed dishonest to call them "events."  So instead, they're "transitions."

The transition ledger is the source of truth about the state of:
- users and their profiles
- rooms, their participants and levels

It is a time series of _transitions_, which are any events that convey a
change in users, user profiles, rooms, or room participants.

In principle a server need store only the ledger, and from it reconstruct the
"current" state of users and rooms.

A transition's primary key is a monotonically increasing "tick," which is
referenced in rows of volatile state and in the message history.

This means that data _cannot_ be sharded across users or across rooms.
If the number of users and/or rooms exceeds that which the server can handle,
then there is no recourse but to stand up another server which does not share
any data with the original server.
```sql
create table Transitions(
    tick integer primary key not null,
    happenedWhen datetime,
    URI text not null,
    valueJSON text not null
);
```

### Volatile State
The volatile state is the mutable "current" snapshot based on the transition
ledger.
```sql
create table Users(
    ID text primary key not null,
    parentID text,  -- Users.ID
    forkedTick integer not null,
    forkedWhen datetime
);

create table UserProfileFields(
    userID text,  -- Users.ID
    fieldID text,  -- UserProfileFieldSpecs.ID
    valueJSON text,
    updatedTick integer not null,
    updatedWhen datetime not null,

    primary key (userID, fieldID)
);

-- UserProfileFieldSpecs is actually not volatile, and cannot be deduced from
-- the transition ledger.  Instead, it's a sort of type definition.  It can
-- be thought of as part of the database schema.
create table UserProfileFieldSpecs(
    fieldID text primary key not null,
    description text not null,
    tischSchema text not null,
    updatedWhen datetime not null
)

create table Rooms(
    ID text primary key not null,
    parentID text,  -- Rooms.ID
    forkedWho text,  -- Users.ID
    forkedTick integer not null,
    forkedWhen datetime
);

create table RoomParticipants(
    roomID text not null, -- Rooms.ID
    userID text not null, -- Users.ID
    level integer,
    participationStatus integer not null, -- ParticipationStatusSpecs.ID
    updatedWho text not null, -- Users.ID
    updatedTick integer not null,
    updatedWhen datetime not null,

    primary key(roomID, userID)
);

create index RoomParticipantsByUserID on RoomParticipants(UserID);

-- ParticipationStatusSpecs is actually not volatile, and cannot be deduced from
-- the transition ledger.  Instead, it's a sort of type definition.  It can
-- be thought of as part of the database schema.
--
-- It has the following values:
--
--     0: Invited
--     1: Entered Room
--     2: Left Room
--
-- The following transitions are allowed:
--
--     0 → 1: Invitation Accepted
--     1 → 2: Left Room
--     2 → 1: Reentered Room
create table ParticipationStatusSpecs(
    ID integer primary key,
    description text not null,
    updatedWhen datetime not null
);
```

### Message History
Messages will outnumber other events by far, so they are stored separately, in
such a way that they may be divided across shards and truncated beyond an age
cutoff.
```sql
create table Messages(
    ID text primary key not null,
    tick integer not null,
    roomID text not null,
    userID text not null,
    postedWhen datetime,
    MIME text not null,
    textValue text not null,
);

create index MessageByRoomIDAndTick on Messages(roomID, tick);
```

Storage
-------
The volatile state is meant to fit on a single node.  It might even be
reconstructed from the transition ledger on startup.

The transition ledger is meant to exist in some time series database, or any
old database if the server is expected to be small.

The message history is meant to exist in some distributed immutable storage,
or any old database if the server is expected to be small, or if an aggressive
message purging policy is in place.

Communication
-------------
Messages and transitions are connected.  Associated with each message is a
"tick," which indicates the place in the transition ledger after which the
message occurred.

Thus, to store a message, you need to know the "most recent" transition tick.
If message posts are handled where transitions are handled, then this is not an
issue.  However, if message posts are handled by a different subsystem, then
that subsystem must have access to a "feed" of ticks coming from the subsystem
that handles transitions.
