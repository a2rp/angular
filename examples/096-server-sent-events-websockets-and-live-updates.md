# Server-Sent Events, WebSockets, and Live Updates - Examples

These examples show two different live-update patterns:

1. **Server-Sent Events (SSE)** for a one-way server-to-browser activity feed.
2. **WebSockets** for a two-way chat connection.

They are teaching examples. Replace the example URLs and message contracts with the API implemented by your server. The server must authenticate clients, authorize each operation, validate message sizes and contents, and define its own persistence and replay rules.

## 1. SSE: a one-way live activity feed

Use SSE when the browser mainly receives updates. The client can send actions through normal HTTP endpoints while the open EventSource connection carries updates back to the browser.

### Expected server response

The endpoint, such as /api/live-updates, must return a long-lived response with the SSE content type. Each event ends with a blank line:

~~~text
HTTP/1.1 200 OK
Content-Type: text/event-stream; charset=utf-8
Cache-Control: no-cache
Connection: keep-alive

id: 42
event: message
data: {"id":"42","text":"Build started","createdAt":"2026-11-26T12:00:00.000Z"}

~~~

The last empty line terminates the event. Keep the JSON data shape consistent with the client parser below. A production server should also decide how event IDs, replay after reconnect, heartbeats, authorization, and proxy buffering work.

### Define and validate the event contract

Network data is untrusted. Parse JSON as unknown and validate each field before showing it.

~~~ts
export interface FeedUpdate {
  id: string;
  text: string;
  createdAt: string;
}

function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

export function parseFeedUpdate(value: unknown): FeedUpdate {
  if (!isRecord(value)) {
    throw new Error('Feed update must be an object.');
  }

  const id = value['id'];
  const text = value['text'];
  const createdAt = value['createdAt'];

  if (
    typeof id !== 'string' ||
    id.length === 0 ||
    id.length > 200 ||
    typeof text !== 'string' ||
    text.length > 2000 ||
    typeof createdAt !== 'string' ||
    Number.isNaN(Date.parse(createdAt))
  ) {
    throw new Error('Feed update has an invalid shape.');
  }

  return { id, text, createdAt };
}
~~~

The limit checks protect the UI from unexpectedly large fields. Apply request and event-size limits on the server too.

### Wrap EventSource in an Observable

The wrapper reports connection changes and updates through one typed stream. Its teardown closes the browser connection when the subscriber unsubscribes.

~~~ts
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';

export type FeedConnection =
  | 'connecting'
  | 'connected'
  | 'reconnecting'
  | 'closed'
  | 'failed';

export type FeedEvent =
  | {
      kind: 'connection';
      status: FeedConnection;
      notice?: string;
    }
  | {
      kind: 'update';
      update: FeedUpdate;
    };

@Injectable({ providedIn: 'root' })
export class LiveFeedService {
  connect(url: string): Observable<FeedEvent> {
    return new Observable<FeedEvent>((subscriber) => {
      if (typeof EventSource === 'undefined') {
        subscriber.next({
          kind: 'connection',
          status: 'failed',
          notice: 'Live updates are unavailable in this environment.'
        });
        subscriber.complete();
        return;
      }

      const source = new EventSource(url);

      subscriber.next({ kind: 'connection', status: 'connecting' });

      source.onopen = () => {
        subscriber.next({ kind: 'connection', status: 'connected' });
      };

      source.onmessage = (event: MessageEvent<string>) => {
        try {
          const value: unknown = JSON.parse(event.data);
          const update = parseFeedUpdate(value);
          subscriber.next({ kind: 'update', update });
        } catch {
          // Ignore one invalid event while keeping the EventSource alive.
          subscriber.next({
            kind: 'connection',
            status: 'connected',
            notice: 'An unreadable feed update was ignored.'
          });
        }
      };

      source.onerror = () => {
        if (source.readyState === EventSource.CONNECTING) {
          // EventSource will attempt its built-in reconnect.
          subscriber.next({
            kind: 'connection',
            status: 'reconnecting'
          });
        } else if (source.readyState === EventSource.CLOSED) {
          subscriber.next({ kind: 'connection', status: 'closed' });
          subscriber.complete();
        }
      };

      return () => source.close();
    });
  }
}
~~~

Do not call subscriber.error for every EventSource error: transient errors can mean that the browser is reconnecting. Ending the Observable would run teardown and close that reconnecting EventSource. The browser's EventSource API handles its reconnect attempts; the application still needs a clear UI state and server-side replay strategy.

Each call to connect creates a separate EventSource. If several components need the same feed, decide deliberately whether to share one connection and how its lifetime should be owned.

### Accumulate a bounded view model in an Angular component

This component keeps connection status separate from received updates, retains at most 50 items, and converts the Observable into a signal. The signal subscription is cleaned up with the component's injection context.

~~~ts
import { DatePipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { catchError, of, scan } from 'rxjs';
import {
  FeedConnection,
  FeedEvent,
  LiveFeedService
} from './live-feed.service';

interface FeedViewModel {
  connection: FeedConnection;
  items: FeedUpdate[];
  lastUpdatedAt?: number;
  notice?: string;
}

const INITIAL_FEED_STATE: FeedViewModel = {
  connection: 'connecting',
  items: []
};

function reduceFeedState(
  state: FeedViewModel,
  event: FeedEvent
): FeedViewModel {
  if (event.kind === 'connection') {
    return {
      ...state,
      connection: event.status,
      notice: event.notice
    };
  }

  return {
    ...state,
    items: [...state.items, event.update].slice(-50),
    lastUpdatedAt: Date.now(),
    notice: undefined
  };
}

@Component({
  selector: 'app-live-feed',
  standalone: true,
  imports: [DatePipe],
  templateUrl: './live-feed.component.html'
})
export class LiveFeedComponent {
  private readonly feed = inject(LiveFeedService);

  readonly state = toSignal(
    this.feed.connect('/api/live-updates').pipe(
      scan(reduceFeedState, INITIAL_FEED_STATE),
      catchError(() =>
        of({
          ...INITIAL_FEED_STATE,
          connection: 'failed' as const,
          notice: 'The live feed could not be opened.'
        })
      )
    ),
    { initialValue: INITIAL_FEED_STATE }
  );
}
~~~

~~~html
<section aria-labelledby="feed-title">
  <h2 id="feed-title">Live activity</h2>

  <p role="status" aria-live="polite">
    Feed status: {{ state().connection }}
  </p>

  @if (state().notice; as notice) {
    <p>{{ notice }}</p>
  }

  @if (state().lastUpdatedAt; as updatedAt) {
    <p>
      Last update received:
      <time>{{ updatedAt | date: 'mediumTime' }}</time>
    </p>
  }

  <ol>
    @for (item of state().items; track item.id) {
      <li>
        <time [attr.datetime]="item.createdAt">
          {{ item.createdAt | date: 'medium' }}
        </time>
        <span>{{ item.text }}</span>
      </li>
    } @empty {
      <li>No activity has arrived yet.</li>
    }
  </ol>
</section>
~~~

Angular interpolation renders message text as text, which is appropriate for untrusted feed content. Do not render server-provided strings with bypassed sanitization or as trusted HTML.

## 2. WebSocket: a two-way chat connection

Use a WebSocket when both sides need to send messages over a persistent connection. This example uses RxJS WebSocketSubject, validates server messages, exposes the latest bounded message list as a signal, and closes the subject with its owning component.

### Define the two directions of the protocol

The browser sends a small send packet. The server sends message packets with an ID, sender, and timestamp.

~~~ts
export interface ChatServerMessage {
  type: 'message';
  id: string;
  sender: string;
  text: string;
  sentAt: string;
}

export interface ChatClientMessage {
  type: 'send';
  text: string;
}

export type ChatPacket = ChatServerMessage | ChatClientMessage;

function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

export function parseChatServerMessage(value: unknown): ChatServerMessage {
  if (
    !isRecord(value) ||
    value['type'] !== 'message' ||
    typeof value['id'] !== 'string' ||
    value['id'].length === 0 ||
    typeof value['sender'] !== 'string' ||
    value['sender'].length > 100 ||
    typeof value['text'] !== 'string' ||
    value['text'].length > 2000 ||
    typeof value['sentAt'] !== 'string' ||
    Number.isNaN(Date.parse(value['sentAt']))
  ) {
    throw new Error('The server sent an invalid chat message.');
  }

  return {
    type: 'message',
    id: value['id'],
    sender: value['sender'],
    text: value['text'],
    sentAt: value['sentAt']
  };
}
~~~

A client-side input limit improves the interface, but the server must enforce its own size, authorization, rate, and content rules. Never assume that a message is allowed just because the socket is connected.

### Create a component-owned WebSocketSubject

The example intentionally has no automatic retry. WebSocket connections do not reconnect automatically in the browser. A real reconnect design should create a fresh subject and define a bounded retry policy, backoff with jitter, cancellation, and how the UI represents stale messages.

~~~ts
import { DatePipe } from '@angular/common';
import { Component, DestroyRef, inject, signal } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { toSignal } from '@angular/core/rxjs-interop';
import {
  EMPTY,
  catchError,
  filter,
  scan
} from 'rxjs';
import { webSocket } from 'rxjs/webSocket';

type ChatConnection = 'connecting' | 'connected' | 'closed' | 'failed';

@Component({
  selector: 'app-live-chat',
  standalone: true,
  imports: [DatePipe, FormsModule],
  templateUrl: './live-chat.component.html'
})
export class LiveChatComponent {
  private readonly destroyRef = inject(DestroyRef);
  private socketFailed = false;

  readonly connection = signal<ChatConnection>('connecting');
  draft = '';

  private readonly socket = webSocket<ChatPacket>({
    url: 'wss://api.example.com/chat',
    serializer: (packet) => JSON.stringify(packet),
    deserializer: (event) => {
      const value: unknown = JSON.parse(String(event.data));
      return parseChatServerMessage(value);
    },
    openObserver: {
      next: () => this.connection.set('connected')
    },
    closeObserver: {
      next: () => {
        if (!this.socketFailed) {
          this.connection.set('closed');
        }
      }
    }
  });

  readonly messages = toSignal(
    this.socket.pipe(
      filter(
        (packet): packet is ChatServerMessage =>
          packet.type === 'message'
      ),
      scan(
        (messages, message) => [...messages, message].slice(-100),
        [] as ChatServerMessage[]
      ),
      catchError(() => {
        this.socketFailed = true;
        this.connection.set('failed');
        return EMPTY;
      })
    ),
    { initialValue: [] as ChatServerMessage[] }
  );

  constructor() {
    this.destroyRef.onDestroy(() => this.socket.complete());
  }

  send(): void {
    const text = this.draft.trim();

    if (
      this.connection() !== 'connected' ||
      text.length === 0 ||
      text.length > 2000
    ) {
      return;
    }

    this.socket.next({ type: 'send', text });
    this.draft = '';
  }
}
~~~

~~~html
<section aria-labelledby="chat-title">
  <h2 id="chat-title">Live chat</h2>

  <p role="status" aria-live="polite">
    Connection: {{ connection() }}
  </p>

  <ol>
    @for (message of messages(); track message.id) {
      <li>
        <strong>{{ message.sender }}</strong>
        <span>{{ message.text }}</span>
        <time [attr.datetime]="message.sentAt">
          {{ message.sentAt | date: 'shortTime' }}
        </time>
      </li>
    } @empty {
      <li>No messages have arrived yet.</li>
    }
  </ol>

  <form (ngSubmit)="send()">
    <label for="chat-message">Message</label>
    <input
      id="chat-message"
      name="message"
      [(ngModel)]="draft"
      maxlength="2000"
      autocomplete="off"
    />
    <button
      type="submit"
      [disabled]="connection() !== 'connected' || !draft.trim()"
    >
      Send
    </button>
  </form>
</section>
~~~

The server should identify the sender and create authoritative message IDs and timestamps. Treat those values as server-owned; do not trust a client-supplied sender or ID.

## 3. Choose the transport for the feature

| Requirement | Starting point |
| --- | --- |
| Occasional refresh with simple caching | HTTP request or polling |
| Frequent updates, mostly server to browser | SSE |
| Interactive messages in both directions | WebSocket |
| Update history after disconnect | Server-supported event IDs, cursors, or a normal HTTP history endpoint |
| Several views consume one feed | Deliberately shared connection with an explicit owner and teardown policy |

SSE is a text event stream with browser-managed reconnect behavior. WebSocket supports two-way messages, but application code must own reconnection behavior. In Angular, toSignal subscribes immediately and cleans up when created in an injection context; takeUntilDestroyed is another option for imperative subscriptions. See the Angular toSignal API, takeUntilDestroyed guide, MDN SSE guide, and RxJS WebSocketSubject configuration.

## 4. Production checklist

- Use HTTPS and WSS in production.
- Authenticate the connection and authorize each action on the server.
- Validate the WebSocket Origin on the server where appropriate.
- Do not put long-lived secrets in URLs; browser EventSource and WebSocket constructors do not provide a general arbitrary-header option.
- Validate every incoming event and message, and bound message size and retained UI history.
- Keep connection status separate from the last known data.
- Decide how reconnecting clients recover missed updates and deduplicate repeated event IDs.
- Define a bounded WebSocket reconnect policy if the product requires automatic recovery.
- Close the connection when its feature owner is destroyed.
- Show whether displayed data may be stale after a disconnect.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="095-angular-resources-and-signal-based-asynchronous-data.md">095 - Angular Resources and Signal-Based Asynchronous Data</a></td><td><a href="../lessons/096-server-sent-events-websockets-and-live-updates.md">Lesson</a></td><td><a href="../qnas/096-server-sent-events-websockets-and-live-updates.md">Q&amp;A</a></td><td><a href="097-feature-based-application-architecture.md">097 - Feature-Based Application Architecture</a></td>
</tr>
  </tbody>
</table>
