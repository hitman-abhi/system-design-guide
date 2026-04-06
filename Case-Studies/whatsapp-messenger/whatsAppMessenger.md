# WhatsApp Messenger

## 1. Problem Statement

Design a large-scale messaging application similar to WhatsApp, with primary focus on:

- one-to-one chat
- group chat
- message delivery states
- read receipts

At small scale, messaging looks deceptively simple:

- sender posts a message
- receiver gets a message
- UI shows the conversation

At large scale, the system becomes much more challenging because messaging combines several hard requirements at once:

- low-latency interactive delivery
- durability
- offline handling
- multi-device behavior
- ordering expectations
- massive fan-out for group chat
- privacy-sensitive metadata handling

The product expectations are also unusually strict.

Users expect:

- messages to appear quickly
- old messages to be available after reconnect
- read receipts to feel accurate enough
- group chats to behave sensibly even under high fan-out

This is why messaging systems are a strong system-design case study.

They force the architecture to handle both:

- real-time paths
- asynchronous recovery paths

without making the product feel inconsistent or fragile.

For this case study, the focus is intentionally not on voice/video calling or media pipelines. The focus is on the core messaging platform for:

- direct chats
- group chats
- read receipts

## 2. Scope and Assumptions

In scope:

- user-to-user text messaging
- group messaging
- message persistence
- delivery to online and offline users
- read receipts
- message ordering within a conversation
- basic sync across reconnects

Out of scope for this version:

- end-to-end encryption protocol details
- voice and video calling
- media upload and CDN pipelines
- ephemeral stories/status features
- advanced anti-spam models

Assumptions:

- most users are mobile users
- the system is internet-scale
- users may be online on multiple devices
- read receipts are important but do not need database-transaction-level global synchrony
- message history is durable

We will assume the design is for a mature production system, not an MVP.

## 3. Functional Requirements

The system must support:

- sending a direct message from one user to another
- receiving direct messages in order within a conversation
- creating a group
- sending messages to a group
- receiving group messages as group members
- showing message delivery state
- showing read receipts
- syncing missed messages when a user reconnects
- fetching conversation history

Secondary functional behaviors:

- group membership changes
- muted or archived conversations are out of scope at the core architecture level but would fit on top
- presence and typing indicators are optional and not central here

## 4. Non-Functional Requirements

Important non-functional requirements:

- low latency for live message delivery
- very high availability
- durable message storage
- conversation-level ordering that is good enough for user expectations
- scalability for hot groups
- tolerance of offline recipients
- efficient fan-out for group delivery
- secure handling of sensitive user communication metadata

Consistency expectations:

- sending a message should not require all recipients to be online
- message history should be durable once accepted
- read receipts can be eventually consistent but should converge quickly
- ordering should be preserved within a conversation as much as possible

The system should prefer:

- availability and durable queued delivery

over:

- strict global synchronization across all devices at send time

## 5. Capacity and Scale Estimation

Assume:

- 500 million daily active users
- 100 billion messages per day across all chats and groups

Average messages per second:

- about 1.16 million messages per second globally

Peak traffic may be 3x to 5x average depending on geography and time-of-day concentration.

Assume peak:

- 4 million to 5 million messages per second globally

Storage estimate:

Suppose average text message payload plus metadata is around 300 bytes to 1 KB depending on message metadata richness.

Using 500 bytes as a rough average:

- 100 billion messages/day -> about 50 TB/day raw logical data

This is before:

- replication
- indexing
- delivery state
- read receipt state

So this is clearly a system where:

- append-heavy storage
- sharding
- cold vs hot retention strategy

all matter.

Connection estimate:

Not every user is online simultaneously.

Assume:

- 50 million to 100 million concurrent connected users at high scale

This implies:

- a very large persistent-connection layer
- gateway fan-out and regional partitioning

Group chat scale:

Most groups are small.

Some groups can be large and hot.

The long tail matters because fan-out cost in group messaging is one of the main scaling pressures.

## 6. Core Data Model

The main entities:

- `User`
- `Conversation`
- `ConversationMember`
- `Message`
- `DeliveryState`
- `ReadState`
- `DeviceSession`

Useful modeling choice:

Treat direct chats and group chats under a shared conversation abstraction.

That means:

- direct chat = conversation with 2 members
- group chat = conversation with N members

Main fields:

### Conversation

- `conversation_id`
- `conversation_type` (`direct`, `group`)
- `created_at`
- group metadata if applicable

### ConversationMember

- `conversation_id`
- `user_id`
- membership role
- joined_at
- left_at if historical membership is retained

### Message

- `message_id`
- `conversation_id`
- `sender_id`
- `server_sequence` or conversation sequence
- `client_message_id` for dedupe
- payload
- created_at

### Delivery State

At minimum, some systems model:

- accepted by server
- delivered to recipient device(s)
- read by recipient(s)

For groups, delivery and read state may need aggregation semantics rather than one row per member on the hot path unless the product explicitly requires per-member receipts.

### Read State

Instead of storing a separate read receipt row for every message per user on the hot path, a common strategy is to store:

- per-user per-conversation read cursor

This is far more scalable than modeling every read as a fully independent record.

That design choice matters a lot and will be discussed in the deep dive.

## 7. APIs or External Interfaces

Core interfaces:

### Send Message

`POST /conversations/{conversation_id}/messages`

Input:

- sender auth
- message payload
- optional client-generated dedupe ID

Output:

- accepted message ID
- server timestamp
- initial delivery state

### Sync Messages

`GET /conversations/{conversation_id}/messages?after_sequence=X`

Used after reconnect or app resume.

### Create Group

`POST /groups`

Input:

- creator
- member list
- group metadata

### Update Read Cursor

`POST /conversations/{conversation_id}/read`

Input:

- last_seen_message_sequence or message ID

Output:

- acknowledgement that read state advanced

### Persistent Connection Channel

In practice, message delivery is usually driven over:

- WebSocket-like persistent connection
- or mobile push + sync fallback

The HTTP endpoints are useful to describe the contract, but the live system will often depend on a persistent session layer for real-time delivery.

#### Why WebSocket-Style Persistent Connections Are a Good Fit

For a large-scale chat system, WebSocket-style persistent connections are usually the best fit for live delivery because messaging is not a simple request-response workload.

The server needs to be able to:

- push new messages to the client immediately
- push read receipt updates
- push typing or presence-style events if supported later
- detect whether the client is still connected

Trying to do this with repeated HTTP polling is usually much worse because:

- latency is higher
- mobile battery usage is worse
- server request overhead is much higher
- the system spends too much effort on asking "anything new?" instead of delivering actual new events

Long polling improves on naive polling, but it still keeps the communication model closer to repeated request cycling than to a true long-lived session.

A WebSocket-style connection is a better fit because it gives the system:

- one long-lived authenticated session
- bidirectional communication
- lower per-message overhead
- better real-time fan-out behavior

That does not mean WebSocket alone solves the entire delivery problem.

Mobile clients still need:

- reconnect logic
- offline sync
- push notification fallback when the app is backgrounded or disconnected

So the practical recommendation is:

- use persistent WebSocket-style sessions for live delivery
- use HTTP-based sync APIs for catch-up and history retrieval
- use push notifications as a wake-up or re-engagement path, not as the durable messaging channel

## 8. High-Level Design

At a large scale, the system should separate:

- connection handling
- message ingestion
- message persistence
- fan-out and delivery
- read state updates

Core components:

- clients
- edge/gateway layer for persistent connections
- load balancer
- chat gateway fleet
- messaging service fleet
- conversation and message store
- fan-out workers
- delivery queue / stream
- read-state service
- notification / offline push service

```mermaid
flowchart LR
    U[Mobile / Web Client] --> EDGE[Edge / connection ingress]
    EDGE --> LB[Load balancer]
    LB --> GW[Chat gateway fleet]
    GW --> MSG[Messaging service fleet]
    MSG --> STORE[(Conversation + message store)]
    MSG --> STREAM[Delivery stream / queue]
    STREAM --> FAN[Fan-out workers]
    FAN --> GW
    FAN --> PUSH[Push notification service]
    GW --> READ[Read-state service]
    READ --> STORE
```

What to notice:

- persistent connection handling is separated from core message persistence logic
- the message write path and delivery fan-out path are distinct
- offline delivery and push notification are secondary paths, not the main synchronous send path
- read-state updates are modeled as their own concern because read receipts evolve independently from message creation

At smaller scale, several of these responsibilities can live in one logical service. At large scale, separating them makes throughput shaping and operational isolation much easier.

### Why Separate Connection Gateways from Messaging Services

Persistent connection handling is a very different workload from message persistence and fan-out.

The gateway tier cares about:

- connection count
- heartbeats
- routing connected users to the right backend

The messaging service tier cares about:

- persistence
- dedupe
- sequence assignment
- conversation authorization

Keeping them separate helps because:

- connection load scales with online users
- write load scales with messages
- deploy risk is isolated
- the system can evolve fan-out logic without rebuilding the connection layer every time

## 9. Request Flows

### Direct Message Send Flow

```mermaid
sequenceDiagram
    participant S as Sender Client
    participant GW as Chat Gateway
    participant MSG as Messaging Service
    participant DB as Message Store
    participant Q as Delivery Stream
    participant RGW as Recipient Gateway

    S->>GW: send message
    GW->>MSG: authenticated send request
    MSG->>DB: persist message and assign sequence
    DB-->>MSG: write success
    MSG->>Q: publish delivery event
    MSG-->>GW: ack accepted
    GW-->>S: message accepted
    Q->>RGW: deliver to recipient if online
    RGW-->>Q: delivery ack
```

What to notice:

- durable persistence happens before the sender sees success
- fan-out is decoupled from message acceptance
- online delivery is fast but not required for send success

### Offline Recipient Flow

```mermaid
sequenceDiagram
    participant S as Sender Client
    participant MSG as Messaging Service
    participant DB as Message Store
    participant Q as Delivery Stream
    participant PUSH as Push Service
    participant R as Recipient Client

    S->>MSG: send message
    MSG->>DB: persist message
    DB-->>MSG: success
    MSG->>Q: publish undelivered event
    Q->>PUSH: trigger push notification
    R->>MSG: reconnect / sync after notification
    MSG->>DB: fetch messages after last sequence
    DB-->>MSG: missed messages
    MSG-->>R: deliver backlog
```

What to notice:

- offline delivery is durability plus later synchronization
- push notification is a hint, not the message transport of record
- sync is essential because push delivery is not a durable messaging layer

### Group Message Flow

```mermaid
sequenceDiagram
    participant S as Sender Client
    participant MSG as Messaging Service
    participant DB as Message Store
    participant Q as Delivery Stream
    participant FAN as Fan-out Workers
    participant GW as Recipient Gateways

    S->>MSG: send group message
    MSG->>DB: persist message in conversation
    DB-->>MSG: success
    MSG->>Q: publish group delivery event
    MSG-->>S: accepted
    Q->>FAN: consume group event
    FAN->>GW: deliver to online members
```

What to notice:

- the sender path should not synchronously wait for delivery to every member
- group fan-out is one of the main scaling pressure points
- persistence and fan-out are intentionally separated

### Read Receipt Flow

```mermaid
sequenceDiagram
    participant R as Recipient Client
    participant GW as Chat Gateway
    participant READ as Read-State Service
    participant DB as Read-State Store
    participant Q as Receipt Event Stream
    participant SGW as Sender Gateway

    R->>GW: mark conversation read up to sequence N
    GW->>READ: advance read cursor
    READ->>DB: update per-user read cursor
    DB-->>READ: success
    READ->>Q: publish read-state update
    Q->>SGW: notify interested sender device(s)
```

What to notice:

- read receipts are best modeled as read cursor advancement, not one independent hot write per message
- sender UI updates can be event-driven
- durable read state and visible receipt propagation can be decoupled

## 10. Deep Dive Areas

### 10.1 Conversation Partitioning and Message Ordering

A messaging system needs some ordering guarantee or the product feels broken quickly.

The strongest useful guarantee is usually:

- preserve ordering within one conversation

Trying to preserve one global order across the whole system is unnecessary and expensive.

The most practical design is to shard by:

- `conversation_id`

That gives the system a clean unit for:

- sequencing
- storage placement
- read and sync access

If each conversation is routed consistently, then assigning a monotonically increasing server sequence within that conversation becomes straightforward.

Potential approaches:

#### Server-Assigned Per-Conversation Sequence

Each persisted message gets a sequence number inside the conversation.

Strengths:

- simple client sync semantics
- easy read-cursor model
- good fit for history retrieval

Costs:

- the write path for one conversation must serialize enough to maintain ordering

#### Client Timestamps

Use sender timestamps as message order.

Strengths:

- very simple conceptually

Costs:

- clocks are unreliable
- retries and reordering make this fragile

This is not sufficient as the canonical order.

**Recommendation**

I would use:

- `conversation_id` as the primary partition key
- server-assigned per-conversation sequence numbers as the canonical order

Why:

- direct chats and groups are naturally conversation-centric
- sync and read receipts become simpler
- user expectation is conversation order, not global order

For very large groups, hot-partition pressure still needs more thought, but this is the clean baseline model.

### 10.2 Message Persistence Model

The system must decide what "message sent successfully" means.

For a mature messaging system, the sender should not see success unless the message is durably persisted by the service.

That means the write path should be:

1. authenticate and authorize sender
2. persist message
3. acknowledge sender
4. asynchronously fan out to recipients

This is better than:

- accept in memory
- try delivery first
- persist later

because crashes would lose messages after the UI already showed success.

Storage considerations:

- append-heavy write path
- conversation-based retrieval
- long retention horizon

Potential storage layout:

- partition by conversation
- sort by conversation sequence
- separate conversation metadata from message body rows

**Recommendation**

I would treat persistence as the acceptance boundary:

- sender ack only after durable write

Why:

- it keeps the user model honest
- it simplifies reconnect sync
- it decouples delivery from correctness

### 10.3 Group Chat Fan-Out Strategy

Group chat is where messaging systems become much harder.

A direct message targets one receiver.

A group message may target:

- tens
- hundreds
- thousands

of members.

There are two broad strategies.

#### Fan-Out on Write

When a group message arrives, the system creates recipient-specific delivery work for group members immediately.

Strengths:

- faster per-recipient retrieval later
- easy push to online members

Costs:

- expensive write amplification
- very hot groups become painful

#### Fan-Out on Read

Persist one canonical group message and let recipients pull or sync from shared conversation history.

Strengths:

- write path stays lighter
- better for very large groups

Costs:

- recipient sync path does more work
- online push delivery still needs recipient targeting

In practice, mature systems often use a hybrid:

- one canonical stored message
- per-recipient delivery work only as needed for online sessions, push notifications, or unread state

**Recommendation**

For WhatsApp-style group chat, I would use:

- canonical message persisted once per conversation
- asynchronous fan-out for online delivery and notifications
- conversation history sync for durability and offline catch-up

Why:

- it avoids storing full message copies for every group member
- it keeps the source of truth clean
- it scales better for larger groups

### 10.4 Read Receipts Modeling

Read receipts can look simple in the UI:

- one check mark
- two check marks
- blue check marks

Internally, there are multiple possible models.

#### Per-Message Per-Recipient Read Rows

For every delivered message, store a read record when each recipient reads it.

Strengths:

- precise

Costs:

- huge write amplification
- especially bad in group chats

#### Per-Conversation Read Cursor

Store, for each user in a conversation:

- highest sequence number read

Strengths:

- compact
- easy to advance
- natural fit for conversation-ordered systems

Costs:

- UI derivation may need logic to map cursor position into message read states

For groups, product semantics matter:

- show read by everyone?
- show some members?
- show only aggregate state?

Those choices drastically affect storage cost.

**Recommendation**

I would model read receipts primarily as:

- per-user per-conversation read cursor

and derive UI receipt states from that.

Why:

- it scales much better
- it fits conversation-sequence ordering
- it makes "mark all up to here as read" a natural operation

For very detailed per-member group read views, I would expose them only if the product truly requires them, because they increase write and storage pressure significantly.

### 10.5 Multi-Device and Offline Sync

Messaging users expect reconnect and device continuity.

That means the system must not rely only on live push over sockets.

Each client should track:

- last synced sequence per conversation

and use sync APIs to fetch:

- missed messages
- updated receipt state

This is also why the system should maintain:

- durable canonical message history

instead of treating delivery queues as the source of truth.

**Recommendation**

I would treat:

- persistent history store
- plus last-seen per-device or per-user sync position

as the foundation of reconnect behavior.

Live sockets improve latency. They do not replace durable sync.

## 11. Bottlenecks and Failure Modes

### Hot Conversations

Some conversations or groups can become extremely hot.

This creates:

- partition hotspots
- fan-out pressure
- gateway delivery bursts

Mitigations:

- partition by conversation but monitor hot-key skew
- isolate very large groups if needed
- use asynchronous fan-out workers

### Gateway Connection Pressure

Persistent connection fleets can be stressed by:

- huge concurrent user counts
- reconnect storms after network outages
- heartbeats and session churn

Mitigations:

- regional connection sharding
- graceful reconnect backoff
- stateless gateways with external session mapping

### Delivery Backlog

If delivery workers or push integrations lag:

- online delivery gets delayed
- offline push may fall behind

Durable message storage keeps correctness intact, but user experience degrades.

### Duplicate Delivery

Retries or reconnect logic can cause duplicate message presentation if the client and server do not agree on idempotent message identity.

Mitigations:

- client-generated message IDs
- server dedupe on resend

### Read Receipt Lag

Receipts are secondary to message durability.

If receipt propagation lags:

- messages are still delivered
- UI state may look temporarily stale

This is acceptable if the system converges quickly and receipts do not block core messaging.

### Membership Churn in Groups

Group membership changes can race with fan-out.

Questions include:

- should a just-removed member receive messages already in flight
- how is group membership snapshotted for fan-out

This needs explicit semantics.

## 12. Scaling Strategy

### Stage 1: Single Region, Simple Chat Service

Start with:

- one messaging service
- one datastore
- one WebSocket gateway layer

This works at moderate scale and helps establish conversation and receipt semantics clearly.

### Stage 2: Separate Gateways, Messaging Core, and Async Fan-Out

As traffic grows:

- split connection handling from persistence
- move delivery to queues and workers
- add dedicated read-state handling

This is usually the first major architectural step.

### Stage 3: Conversation Sharding

Shard message storage by conversation ID.

This gives a scalable routing and retrieval model for most chat workloads.

### Stage 4: Regionalization

As users become global:

- route clients to nearby gateway regions
- keep messaging storage sharded regionally or globally depending product semantics
- replicate enough metadata for routing and continuity

### Stage 5: Large Group Optimizations

If very large groups become important:

- treat large groups as special fan-out workloads
- optimize push and sync behavior separately
- consider specialized pipelines for group delivery rather than treating them like normal 1:1 chats

## 13. Tradeoffs and Alternatives

### Strong Ordering vs Availability

Stronger per-conversation ordering is valuable.

Global ordering is unnecessary and too expensive.

### Write-Time Fan-Out vs Read-Time Reconstruction

Write-time fan-out improves live delivery convenience but increases write amplification.

Canonical store plus async fan-out is usually the more scalable balance.

### Per-Message Read Records vs Read Cursor

Per-message receipts are precise and expensive.

Read cursors are compact and practical.

For large-scale messaging, cursors are usually the better default.

### One Service vs Several Services

A smaller system can combine:

- gateway
- persistence
- fan-out

At large scale, separating them helps with:

- connection scaling
- deployment safety
- queue-based decoupling

## 14. Real-World Considerations

### Abuse and Spam

Messaging systems need controls for:

- spam bursts
- bot behavior
- invite abuse
- large-group abuse

### Privacy and Security

Even without going deep into encryption design, the system must treat messaging metadata carefully:

- who talks to whom
- when messages are read
- device and session information

### Observability

Important metrics:

- send latency
- durable write latency
- fan-out lag
- offline sync latency
- push notification success
- read receipt lag
- reconnect storm impact

### Product Semantics

Messaging systems are highly sensitive to user-visible semantics.

The system must define:

- what "sent" means
- what "delivered" means
- what "read" means
- what happens if a user has multiple devices

If these are vague internally, the product experience becomes inconsistent.

## 15. Summary

A WhatsApp-style messenger is fundamentally a conversation-centric distributed system with two dominant pressures:

- low-latency live delivery
- durable asynchronous recovery

The architecture should therefore center around:

- persistent connection gateways
- durable message persistence before sender acknowledgment
- conversation-based partitioning
- asynchronous fan-out
- compact read-state modeling through read cursors

The hardest parts are not only sending a message.

They are:

- scaling group delivery
- preserving ordering where it matters
- syncing offline users
- modeling read receipts efficiently

That is why a good messaging design is less about one "chat service" box and more about explicitly separating:

- connection handling
- persistence
- fan-out
- read-state propagation

while keeping the user experience coherent.
