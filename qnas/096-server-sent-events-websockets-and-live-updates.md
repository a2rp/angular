# Server-Sent Events, WebSockets, and Live Updates: Q&A

## 1. What is a live update connection?

It is a connection that lets the server deliver new information without requiring the client to make a fresh request for every update. The application still needs to decide how to represent the connection and its data state.

## 2. Should every changing view use a persistent connection?

No. If updates are infrequent or a short delay is acceptable, ordinary HTTP requests or polling may be simpler and easier to operate.

## 3. What is the main difference between polling and server push?

With polling, the client periodically asks whether anything changed. With server push, the client opens a longer-lived connection and the server sends data when it is available.

## 4. What is Server-Sent Events?

SSE is a browser API and HTTP event-stream format for one-way server-to-client updates. The browser uses EventSource to receive messages from a text/event-stream response.

## 5. What is the direction of an SSE connection?

The server sends events to the browser. The client can send an action through a separate HTTP request. EventSource itself is not a two-way messaging channel.

## 6. What content type does an SSE endpoint use?

It should respond with text/event-stream. Events are UTF-8 text blocks separated by a blank line.

## 7. What do the event, id, and data fields do?

event names a custom event type, id identifies an event for reconnect handling, and data carries the message text. Messages without an event field use the default message event.

## 8. How does an SSE event end?

A blank line ends an event. An unfinished final block is not necessarily dispatched until the server sends the required blank line.

## 9. Does EventSource reconnect automatically?

For temporary connection loss, the browser normally attempts to reconnect. The server can use the retry field to suggest a reconnection delay. The client can stop a connection with close().

## 10. What is Last-Event-ID used for?

When the stream reconnects, the browser can send the last received event ID to the server. The server can use it as a cursor to resume the stream, if it retains and implements replay.

## 11. Does automatic SSE reconnect guarantee that no event is lost?

No. The browser can reconnect, but the server must decide whether and how to replay missed events. If correctness depends on complete delivery, use event IDs, durable replay, acknowledgements, or a later state refresh.

## 12. Can an SSE client send custom authorization headers?

The native EventSource API does not provide a general custom-header option. Depending on the security design, use same-origin cookies, credentialed connections, or a different transport that supports the API contract. Do not put access tokens in URLs.

## 13. What does withCredentials do for EventSource?

It requests that credentials be included in the connection request. Cross-origin use still requires a compatible server CORS and credential configuration. Do not enable credentials without reviewing the origin and cookie policy.

## 14. What is a WebSocket?

A WebSocket provides a persistent connection that can carry messages from client to server and server to client. It can carry text or binary messages.

## 15. When should I prefer WebSockets over SSE?

Use WebSockets when the feature truly needs frequent two-way messages, such as interactive chat or collaboration. For a server-only feed, SSE may be simpler.

## 16. Does a browser WebSocket reconnect automatically?

No. The application must create a new connection and define the retry policy. Use bounded backoff with jitter, retry only failures that make sense, and provide a visible failure state.

## 17. Should every disconnect be retried?

No. A temporary network loss may be retryable. Invalid configuration, expired credentials, denied access, incompatible protocol, or a permanent server response may require another action.

## 18. What does wss mean?

It is WebSocket over a secure transport. Production pages served over HTTPS should use wss for their WebSocket connection.

## 19. Can browser WebSocket connections set arbitrary headers?

The native browser WebSocket constructor does not expose a general custom-header argument. Select an authentication design supported by the browser API and server, and avoid placing secrets in URLs.

## 20. Why should the server validate WebSocket Origin?

Origin validation can help reject unexpected browser origins and reduce cross-site WebSocket attacks. Origin is not an authentication credential because non-browser clients can forge it.

## 21. What is RxJS WebSocketSubject?

It is an RxJS abstraction that acts like an Observable for incoming socket messages and an Observer for outgoing messages. It also supports serializer, deserializer, open, and close hooks.

## 22. Does WebSocketSubject validate JSON?

No. Its default deserializer parses JSON, which checks syntax only. Validate the parsed object shape and allowed values before using them.

## 23. What should a reconnect policy define?

Define retryable errors, retry limits, backoff and jitter, authentication refresh, missed-event recovery, duplicate handling, and what to do with messages queued during disconnect.

## 24. What is backpressure in a live UI?

It is the situation where messages arrive faster than the application can process or render them. Consider batching, limiting history, server-side filtering, or dropping obsolete snapshots.

## 25. How should a live UI track connection and data state?

Keep them separate. A feed can be connected but have no messages yet, or disconnected while still showing previously received data. Use an explicit connection status and data model.

## 26. How do I clean up EventSource?

Call close() when the feature no longer owns the connection. If it is wrapped in an Observable, return teardown logic that closes it when the subscriber unsubscribes.

## 27. How do I clean up a component-owned Observable subscription?

Use the async pipe, toSignal in an injection context, or takeUntilDestroyed for an imperative subscription. Pick one owner and make its lifetime clear.

## 28. Does toSignal subscribe only when a template reads the signal?

No. toSignal subscribes immediately when created. Reuse the signal it returns instead of calling toSignal repeatedly from methods, computed callbacks, or templates.

## 29. What happens if the Observable passed to toSignal errors?

Reading the resulting signal throws that error. Handle expected connection failures before toSignal, or emit a value that represents the error state.

## 30. Does a signal retain every live message?

No. A signal holds the latest value assigned to it. To keep history, accumulate messages explicitly with an RxJS operator such as scan or in a state store, and set a retention limit.

## 31. Should every message be stored in the browser?

Not necessarily. A feed may only need the latest value or a bounded history. If users must not miss events, use durable server storage and an acknowledgement or replay design.

## 32. How should live message payloads be rendered?

Validate payloads and render user-visible text through Angular's normal text interpolation. Do not inject message contents as HTML.

## 33. How should a server protect a live connection?

Authenticate the connection, authorize subscriptions and actions, validate every message, limit message size and rate, and log safely. A successful handshake does not authorize every later operation.

## 34. Can multiple components each create their own connection?

They can, but this may multiply server load and duplicate events. If they need the same feed, give one service ownership and share the stream intentionally.

## 35. When is polling still the right choice?

Polling can be appropriate when data changes rarely, a small delay is acceptable, or persistent connections are difficult to support in the deployment environment. Choose based on the feature and operational constraints.

## 36. What is the main takeaway?

Use polling for simple low-frequency refresh, SSE for one-way server updates, and WebSockets for genuine two-way messaging. Bridge transport events into Angular through Observables or Signals, validate messages, and design cleanup, reconnect, and data-retention behavior explicitly.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="095-angular-resources-and-signal-based-asynchronous-data.md">095 - Angular Resources and Signal-Based Asynchronous Data</a></td><td><a href="../lessons/096-server-sent-events-websockets-and-live-updates.md">Lesson</a></td><td><a href="../examples/096-server-sent-events-websockets-and-live-updates.md">Example</a></td><td><a href="097-feature-based-application-architecture.md">097 - Feature-Based Application Architecture</a></td>
</tr>
  </tbody>
</table>
