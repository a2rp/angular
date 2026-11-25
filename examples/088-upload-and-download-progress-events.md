# Upload and Download Progress Events — Examples

These examples use the current Angular options reportUploadProgress and reportDownloadProgress. They assume HttpClient is provided through provideHttpClient. For older Angular versions, check the version-specific HTTP guide because progress option names may differ.

## 1. Configure XHR for browser upload progress

The default Fetch backend does not support upload progress. Configure withXhr when a browser feature needs it:

~~~ts
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withXhr } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withXhr())
  ]
};
~~~

This configures the application HttpClient backend. Angular recommends the default Fetch backend for server-side rendering, so consider the application's SSR configuration before selecting XHR globally.

## 2. Calculate a determinate percentage safely

The total size may not be available:

~~~ts
export function progressPercent(
  loaded: number,
  total: number | undefined
): number | null {
  if (total === undefined || total <= 0) {
    return null;
  }

  return Math.min(100, Math.round((loaded / total) * 100));
}
~~~

A null result means the UI should use an indeterminate progress state.

## 3. Upload a file and expose typed progress states

~~~ts
// file-upload.service.ts
import {
  HttpClient,
  HttpEventType,
  HttpResponse,
  HttpUploadProgressEvent
} from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { filter, map, Observable } from 'rxjs';

export interface UploadResult {
  id: string;
  fileName: string;
}

export type UploadState =
  | {
      kind: 'uploading';
      loaded: number;
      total?: number;
      percent: number | null;
    }
  | {
      kind: 'complete';
      result: UploadResult;
    };

@Injectable({ providedIn: 'root' })
export class FileUploadService {
  private readonly http = inject(HttpClient);

  upload(file: File): Observable<UploadState> {
    const formData = new FormData();
    formData.append('file', file, file.name);

    const events$ = this.http.post<UploadResult>(
      '/api/files',
      formData,
      {
        observe: 'events',
        reportUploadProgress: true
      }
    );

    return events$.pipe(
      filter(
        (event): event is HttpUploadProgressEvent | HttpResponse<UploadResult> =>
          event.type === HttpEventType.UploadProgress
          || event.type === HttpEventType.Response
      ),
      map(event => {
        if (event.type === HttpEventType.UploadProgress) {
          return {
            kind: 'uploading' as const,
            loaded: event.loaded,
            total: event.total,
            percent: progressPercent(event.loaded, event.total)
          };
        }

        if (event.body === null) {
          throw new Error('The server returned no upload result.');
        }

        return {
          kind: 'complete' as const,
          result: event.body
        };
      })
    );
  }
}
~~~

The service emits uploading states and emits complete only after the final successful response. An HTTP error is delivered through the Observable error channel.

The example uses progressPercent from the preceding section. Keep FormData's Content-Type under browser control so it includes the multipart boundary.

## 4. Display upload state in a component

A component can represent idle, active, complete, and failed states:

~~~ts
// file-upload.component.ts
import { Component, DestroyRef, inject, signal } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { FileUploadService, UploadState } from './file-upload.service';

type UploadUiState =
  | { kind: 'idle' }
  | UploadState
  | { kind: 'error'; message: string };

@Component({
  selector: 'app-file-upload',
  standalone: true,
  templateUrl: './file-upload.component.html'
})
export class FileUploadComponent {
  private readonly uploader = inject(FileUploadService);
  private readonly destroyRef = inject(DestroyRef);
  readonly state = signal<UploadUiState>({ kind: 'idle' });

  selectFile(event: Event): void {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];

    if (!file) {
      return;
    }

    this.state.set({ kind: 'idle' });

    this.uploader.upload(file).pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe({
      next: state => this.state.set(state),
      error: () => {
        this.state.set({
          kind: 'error',
          message: 'The upload failed. Please try again.'
        });
      }
    });
  }
}
~~~

~~~html
<!-- file-upload.component.html -->
<input type="file" (change)="selectFile($event)" />

@if (state(); as current) {
  @if (current.kind === 'uploading') {
    @if (current.percent !== null) {
      <progress aria-label="File upload progress" [value]="current.percent" max="100"></progress>
      <p>Uploading: {{ current.percent }}%</p>
    } @else {
      <progress aria-label="File upload progress"></progress>
      <p>Uploading…</p>
    }
  }

  @if (current.kind === 'complete') {
    <p>Uploaded {{ current.result.fileName }}.</p>
  }

  @if (current.kind === 'error') {
    <p role="alert">{{ current.message }}</p>
  }
}
~~~

For a production component, also manage subscription lifetime if the component can be destroyed during an upload. Unsubscribing can abort the client request, but does not guarantee that a server which already received the file deletes it.

## 5. Request download progress for a Blob

Download progress works with the default Fetch backend:

~~~ts
import {
  HttpClient,
  HttpEvent
} from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ReportDownloadService {
  private readonly http = inject(HttpClient);

  downloadReport(): Observable<HttpEvent<Blob>> {
    return this.http.get('/api/reports/monthly.pdf', {
      observe: 'events',
      responseType: 'blob',
      reportDownloadProgress: true
    });
  }
}
~~~

The stream can include Sent, ResponseHeader, DownloadProgress, and the final Response event. Check event.type before reading progress or response fields.

## 6. Consume download progress and wait for completion

~~~ts
import { HttpEventType } from '@angular/common/http';

 // This code runs in a component with ReportDownloadService injected and
 // downloadMessage and reportBlob signals declared.
this.reportDownload.downloadReport().subscribe({
  next: event => {
    if (event.type === HttpEventType.DownloadProgress) {
      const percent = progressPercent(event.loaded, event.total);

      if (percent === null) {
        this.downloadMessage.set('Downloading report…');
      } else {
        this.downloadMessage.set('Downloading report: ' + percent + '%');
      }
    }

    if (event.type === HttpEventType.Response) {
      if (event.body === null) {
        this.downloadMessage.set('The server returned an empty file.');
        return;
      }

      this.downloadMessage.set('Download complete.');
      this.reportBlob.set(event.body);
    }
  },
  error: () => {
    this.downloadMessage.set('The download failed.');
  }
});
~~~

This stores the completed Blob for another part of the UI to save or preview. Do not mark the operation complete merely because the progress percentage reached 100.

## 7. Narrow event types with RxJS

For reusable pipelines, use a type predicate to narrow an event stream:

~~~ts
const progressEvents$ = events$.pipe(
  filter(
    (event): event is HttpDownloadProgressEvent =>
      event.type === HttpEventType.DownloadProgress
  )
);

progressEvents$.subscribe(event => {
  const percent = progressPercent(event.loaded, event.total);
});
~~~

Similarly, narrow the final response:

~~~ts
const responses$ = events$.pipe(
  filter(
    (event): event is HttpResponse<Blob> =>
      event.type === HttpEventType.Response
  )
);
~~~

Import the event and operator types in the module where the pipeline is defined. Type predicates keep event-specific fields safe to use.

## 8. Keep determinate and indeterminate states accessible

A native progress element can represent known progress:

~~~html
<progress [value]="percent" max="100"></progress>
<p>Downloading report: {{ percent }}%</p>
~~~

For unknown total, omit value:

~~~html
<progress></progress>
<p>Downloading report…</p>
~~~

Use a useful text label and announce major state changes such as started, completed, or failed. Avoid announcing every progress event because updates can be very frequent.

## 9. Older Angular option shape

Older Angular versions may expose the combined reportProgress option:

~~~ts
this.http.post<UploadResult>('/api/files', formData, {
  observe: 'events',
  reportProgress: true
});
~~~

On those versions, reportProgress asks for progress events generally. Upload still requires a backend that supports it; the Fetch limitation remains. For current Angular, prefer reportUploadProgress or reportDownloadProgress according to the transfer direction.

## Quick checklist

- Enable observe: 'events'.
- Turn on only the progress direction the UI needs.
- Use withXhr for upload progress with current Angular.
- Treat total as optional.
- Narrow events by HttpEventType.
- Wait for HttpEventType.Response before showing success.
- Handle failures through the Observable error callback.
- Let the browser set FormData's multipart Content-Type.
- Use accessible labels and avoid excessive announcements.

## Related files and official documentation

- Lesson: lessons/088-upload-and-download-progress-events.md
- Q&A: qnas/088-upload-and-download-progress-events.md
- Angular HTTP request events: https://angular.dev/guide/http/making-requests
- HttpEventType: https://angular.dev/api/common/http/HttpEventType
- Upload progress event: https://angular.dev/api/common/http/HttpUploadProgressEvent
- Download progress event: https://angular.dev/api/common/http/HttpDownloadProgressEvent
- withXhr: https://angular.dev/api/common/http/withXhr
