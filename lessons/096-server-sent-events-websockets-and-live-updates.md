# Server-Sent Events, WebSockets, and Live Updates

A normal HTTP request usually returns a response and ends. Live features keep a connection open so a server can send new information without waiting for the user to refresh. Angular can render those messages through Observables or Signals, but the browser connection, message format, reconnection policy, and security still need deliberate design.

## Learning outcomes

After this lesson, you should be able to:

- Choose polling, Server-Sent Events, or WebSockets for a feature.
- Explain the one-way SSE model and bidirectional WebSocket model.
- Recognize the basic SSE event format and named events.
- Wrap a browser connection in an Observable with correct teardown.
- Convert an Observable to a signal or consume it with the async pipe.
- Handle connection state separately from message data.
- Plan reconnection, duplicate handling, validation, and backpressure.
- Close connections when their owning feature is destroyed.

## 1. Decide whether the feature needs a live connection

Some interfaces only need to refresh occasionally. A normal HTTP request or scheduled polling may be simpler and easier to cache. A persistent connection is useful when new information should appear quickly and repeatedly.

| Approach | Direction | Good fit | Tradeoffs |
| --- | --- | --- | --- |
| Polling with HTTP | Client requests updates | Infrequent refresh or simple infrastructure | Repeats requests when nothing changed; update delay depends on interval |
| Server-Sent Events (SSE) | Server to client | Notifications, progress, feeds, dashboards | Text event stream; client sends actions through separate HTTP requests |
| WebSocket | Client and server in both directions | Chat, collaborative editing, frequent interactive messages | Both sides manage a persistent bidirectional connection and its protocol |

Prefer the least complex approach that meets the product's latency and interaction requirements. Do not add a socket just because the UI displays data that changes.

## 2. How Server-Sent Events work

SSE uses the browser's EventSource API to keep an HTTP response open. The server sends UTF-8 text using the text/event-stream content type. Each event ends with a blank line.

~~~text
event: order-update
id: 842
data: {"orderId":"o-17","status":"shipped"}

~~~

The blank line ends the event. The event field names the event type, id can identify it for reconnect handling, and data carries text. Multiple data lines in one event are joined with newline characters. A message without an event field is delivered as a message event.

The browser can reconnect an EventSource after a temporary connection loss. The server can include an id so the browser can send a Last-Event-ID value when reconnecting. Reconnection does not by itself guarantee that the application has processed every event exactly once; the server and client still need a replay and deduplication policy if gaps matter.

The EventSource constructor accepts a URL and an optional withCredentials setting. Its standard browser API does not provide a general way to attach arbitrary authorization headers. Cookie-based same-origin or credentialed connections may fit some applications. If the API requires a bearer header or a request body, choose an approach that supports that contract instead of putting secrets in a URL.

## 3. How WebSockets work

WebSockets establish a persistent connection that can send messages in either direction. After the opening HTTP handshake, the client and server exchange text or binary messages until one side closes the connection.

Use wss from an HTTPS application. A browser WebSocket client can send and receive messages, but the browser API does not offer arbitrary custom request headers in its constructor. The server should authenticate and authorize the connection, validate its Origin where appropriate, and validate every message. A successful connection is not proof that the client may perform every action.

Unlike EventSource, the browser WebSocket API does not automatically reconnect after a disconnect. A client that needs reconnection must create a new connection and define a bounded policy, such as backoff with jitter, retry limits, and a way for the user to see an unavailable state.

## 4. Keep connection state separate from message data

A live view has at least two dimensions:

1. Connection state: connecting, connected, reconnecting, closed, or failed.
2. Data state: the latest update or the collection of updates already received.

A connection can be healthy and have no data yet. It can also be disconnected while still displaying the last known data. Avoid using an empty message list to mean both "nothing has arrived" and "the connection failed."

A simple view model can express both:

~~~ts
interface LiveFeedState<T> {
  connection: 'connecting' | 'connected' | 'reconnecting' | 'closed' | 'failed';
  items: T[];
  lastUpdatedAt?: number;
  message?: string;
}
~~~

Choose whether a reconnect keeps old content visible. If it does, label it appropriately when freshness matters.

## 5. Wrap a connection in an Observable

A browser connection is an external event source. RxJS can adapt it to an Observable. The Observable teardown must remove listeners and close the connection when the subscriber unsubscribes.

A simplified EventSource wrapper looks like this:

~~~ts
import { Observable } from 'rxjs';

interface FeedMessage {
  id: string;
  text: string;
}

function parseFeedMessage(value: unknown): FeedMessage {
  if (typeof value !== 'object' || value === null) {
    throw new Error('The feed message is invalid.');
  }

  const record = value as Record<string, unknown>;
  const id = record['id'];
  const text = record['text'];

  if (typeof id !== 'string' || typeof text !== 'string') {
    throw new Error('The feed message is invalid.');
  }

  return { id, text };
}

function openFeed(url: string): Observable<FeedMessage> {
  return new Observable<FeedMessage>((subscriber) => {
    const source = new EventSource(url);

    source.onmessage = (event: MessageEvent<string>) => {
      try {
        const body: unknown = JSON.parse(event.data);
        subscriber.next(parseFeedMessage(body));
      } catch {
        subscriber.error(new Error('A feed message could not be read.'));
      }
    };

    source.onerror = () => {
      // EventSource may be reconnecting. Do not error the Observable here
      // unless the application intentionally wants to end that connection.
      if (source.readyState === EventSource.CLOSED) {
        subscriber.complete();
      }
    };

    return () => source.close();
  });
}
~~~

The JSON parse and parser validate a message before it reaches the view. In this simplified wrapper, malformed JSON errors the Observable and therefore ends this subscription; its teardown closes EventSource. If a feed should continue after a malformed event, catch that event locally, report a safe diagnostic, and keep the connection alive.

A transient EventSource error is different: the browser may still be attempting its built-in reconnect. Do not convert every error event into subscriber.error(), because the Observable would terminate and its teardown would close the EventSource. Model reconnecting status separately if the UI needs to display it.

## 6. Bridge an Observable to Angular

There are several ways to consume an Observable in Angular:

- Use the async pipe in a template. It subscribes while the view needs the value and cleans up when the view is destroyed.
- Use toSignal in a component or service when template or computed logic should read the latest emitted value as a signal.
- Use takeUntilDestroyed for an imperative subscription owned by a component or directive.

For example, a service might expose an Observable:

~~~ts
@Injectable({ providedIn: 'root' })
export class FeedService {
  updates(): Observable<FeedMessage> {
    return openFeed('/api/live-updates');
  }
}
~~~

A component can retain the latest value:

~~~ts
import { Component, inject } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-live-feed',
  standalone: true,
  templateUrl: './live-feed.component.html'
})
export class LiveFeedComponent {
  private readonly feed = inject(FeedService);

  readonly latestUpdate = toSignal(this.feed.updates(), {
    initialValue: undefined
  });
}
~~~

~~~html
@if (latestUpdate(); as update) {
  <p>{{ update.text }}</p>
} @else {
  <p role="status">Waiting for updates...</p>
}
~~~

toSignal subscribes immediately when it is created. In an injection context, Angular cleans up that subscription when its component or service is destroyed. Create it once and reuse the returned signal; do not call toSignal repeatedly from a method, computed callback, or template.

A signal represents the latest value. If every message matters, accumulate messages before converting the stream, for example with scan, or store them in a service with a clear retention limit. A UI that retains every update forever can grow without bound.

If an Observable error reaches toSignal, reading the signal throws. Handle expected stream failures before converting them, or model connection and error state as values in the Observable.

## 7. Use RxJS WebSocketSubject for bidirectional messages

RxJS provides webSocket from rxjs/webSocket. It creates a WebSocketSubject: an Observable for messages arriving from the server and an Observer for messages sent to the server.

~~~ts
import { webSocket } from 'rxjs/webSocket';

interface ChatMessage {
  kind: 'chat';
  text: string;
}

const socket = webSocket<ChatMessage>({
  url: 'wss://api.example.com/chat',
  serializer: (message) => JSON.stringify(message),
  deserializer: (event) => JSON.parse(event.data) as ChatMessage
});

// Incoming server messages:
const incoming$ = socket.asObservable();

// Outgoing client message:
socket.next({ kind: 'chat', text: 'Hello' });

// Close the connection through the subject:
socket.complete();
~~~

The default serializer and deserializer use JSON. A TypeScript assertion after JSON.parse is not runtime validation; validate server messages before trusting their fields.

Keep one socket per intended connection scope. If several components need the same connection, put ownership in a shared service rather than constructing a new socket in each view. Decide who can send messages, how the service reports connection state, and what happens to queued actions during a disconnect.

Unsubscribing or completing the connection closes resources, but it does not provide an application-specific reconnect policy. A reconnect normally needs a new WebSocketSubject, appropriate retry timing, an authentication refresh plan, and rules for replaying or discarding unsent messages.

## 8. Reconnect and recover deliberately

A disconnect can happen during a deployment, device sleep, network change, proxy timeout, or server restart. Treat it as a normal state transition.

A robust reconnect policy should answer:

- Which close codes or failures are retryable?
- How many attempts are allowed, and when should retries stop?
- What backoff and random jitter avoid all clients retrying simultaneously?
- How does the UI announce reconnecting or a permanent failure?
- Does the server replay missed events, and how is the cursor stored?
- Are duplicate event IDs ignored safely?
- What happens to client messages that were queued or sent just before disconnect?
- Can the user retry manually or continue using stale data?

SSE already has browser-managed reconnection behavior. You still need to decide how the server resumes events, how long it retains them, and how the application handles authorization failures or a permanently closed source.

Do not blindly retry every error forever. A bad URL, expired credentials, denied permission, or incompatible protocol may need a different action from a short network interruption.

## 9. Validate, bound, and render live data safely

Treat every message as untrusted input, even when it comes from your own backend. Validate its shape, size, identifier, and allowed values before using it. Render text through Angular's normal interpolation. Do not inject message strings as HTML.

Live feeds may be faster than the screen can render. Consider:

- Keeping only the most recent N entries.
- Batching updates before rendering.
- Using stable IDs in @for tracking.
- Dropping obsolete state snapshots while preserving important events.
- Applying server-side filters so the client receives only useful data.
- Measuring message size, rate, and connection count.

If users must see every event, use durable storage or an acknowledgement/replay design rather than relying only on an in-memory browser list.

## 10. Authentication, transport, and server responsibilities

Use HTTPS with SSE and WSS with WebSockets in production. Configure cross-origin policy deliberately. Authenticate the connection and authorize each subscription or action on the server.

Do not place access tokens in query strings; URLs can appear in logs, browser history, monitoring systems, and referrer data. Native EventSource does not accept arbitrary headers. Browser WebSocket connections also have restrictions on custom headers; choose cookie/session authentication, a carefully designed short-lived token exchange, or another supported transport pattern that fits the threat model.

Validate WebSocket Origin on the server when appropriate, but do not treat Origin as an authentication credential. Non-browser clients can forge it. Apply message-size limits, rate limits, authorization checks, and safe logging on the server.

## Common mistakes

### Choosing WebSockets for one-way updates by default

If the client only receives occasional updates, SSE or polling may be simpler. Choose the transport from the required direction and behavior.

### Treating SSE as a two-way channel

EventSource receives server messages. Send user actions through a separate HTTP endpoint unless the feature needs a bidirectional protocol.

### Closing an SSE stream on every error event

EventSource uses error events for connection trouble and may reconnect automatically. Keep transient errors as connection-state updates if the stream should recover.

### Assuming reconnect means no messages were missed

Reconnect only re-establishes transport. Use IDs, replay windows, acknowledgements, or refresh requests if missing events would harm correctness.

### Updating a signal with an unbounded event history

A live feature can retain data for hours. Use a retention policy or persist the history outside the component.

### Forgetting cleanup

Close EventSource and WebSocket connections when their owner ends. If an Observable wraps the connection, put cleanup in its teardown and ensure subscriptions are destroyed.

### Trusting parsed JSON

JSON.parse only checks syntax. Validate the resulting shape and allowed values before using them.

### Reconnecting in a tight loop

Immediate retries can overload the server and clients. Use bounded exponential backoff with jitter and stop when the failure is not retryable.

### Trusting the connection instead of authorizing each action

A valid connection can still send an unauthorized message. Enforce permissions on the server for every action or subscription.

## Key idea

Use SSE for server-to-client updates over HTTP, WebSockets for genuine bidirectional messaging, and polling for simpler low-frequency refresh. In Angular, adapt connection events to Observables or Signals, model connection state separately from data, validate every message, and make cleanup and recovery part of the design.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="095-angular-resources-and-signal-based-asynchronous-data.md">095 - Angular Resources and Signal-Based Asynchronous Data</a></td><td><a href="../qnas/096-server-sent-events-websockets-and-live-updates.md">Q&amp;A</a></td><td><a href="../examples/096-server-sent-events-websockets-and-live-updates.md">Example</a></td><td><a href="097-feature-based-application-architecture.md">097 - Feature-Based Application Architecture</a></td>
</tr>
  </tbody>
</table>
