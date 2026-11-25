# Upload and Download Progress Events

HttpClient normally emits the response body, or the full response when requested. For long transfers, an application can observe the HTTP event stream and report progress while a request body uploads or a response body downloads.

Progress reporting has runtime cost and should be enabled only when the UI uses it. Current Angular request options use reportUploadProgress and reportDownloadProgress to opt into each direction separately.

## Learning outcomes

After this lesson, you should be able to:

- Request an HttpClient event stream with observe: 'events'.
- Enable upload and download progress independently.
- Read loaded bytes and handle a missing total.
- Distinguish progress events from the final HttpResponse.
- Choose a backend that supports upload progress.
- Keep progress UI separate from request completion and error handling.

## 1. HTTP event streams

With observe: 'events', HttpClient emits an Observable of HTTP lifecycle events. Depending on the request and enabled features, the stream can include:

| Event type | Meaning |
| --- | --- |
| Sent | Request has been dispatched |
| UploadProgress | Request body is being uploaded |
| ResponseHeader | Response status and headers have arrived |
| DownloadProgress | Response body is being downloaded |
| Response | Full response, including its body, has arrived |
| User | A custom event from an interceptor or backend |

Events use a type property. Check it before reading fields specific to an event.

## 2. Enable progress deliberately

Progress options are disabled by default because progress events can be frequent and have performance costs.

~~~ts
this.http.get('/api/archive', {
  observe: 'events',
  responseType: 'blob',
  reportDownloadProgress: true
});
~~~

Enable only the progress direction the feature needs. Avoid turning progress reporting on for every request when most callers ignore it.

### Version compatibility

Current Angular HTTP options provide reportUploadProgress and reportDownloadProgress; the older reportProgress option is deprecated in current documentation. Older Angular versions may use reportProgress: true to request progress events. Follow the API for the Angular version installed in the application.

## 3. Upload progress and backend support

The default Fetch backend does not support upload progress. In current Angular, enabling reportUploadProgress with Fetch throws an error. If upload progress is required, configure HttpClient with withXhr():

~~~ts
import { provideHttpClient, withXhr } from '@angular/common/http';

provideHttpClient(withXhr())
~~~

XHR is intended for the browser upload case. Angular recommends the default Fetch backend for server-side rendering, so consider whether progress is necessary in every rendering environment before changing the application-wide backend.

## 4. Read uploaded and downloaded bytes

Progress events report loaded bytes. The total field is optional because the size might not be known:

~~~ts
function getPercent(loaded: number, total: number | undefined): number | null {
  if (total === undefined || total === 0) {
    return null;
  }

  return Math.round((loaded / total) * 100);
}
~~~

When total is absent, show an indeterminate progress indicator rather than displaying a misleading percentage.

## 5. Upload a file using FormData

FormData is commonly used to submit a file:

~~~ts
const formData = new FormData();
formData.append('file', file, file.name);

return this.http.post<UploadResult>('/api/files', formData, {
  observe: 'events',
  reportUploadProgress: true
});
~~~

Do not manually set Content-Type to multipart/form-data for FormData in a browser. The browser must set the multipart boundary parameter.

The final Response event carries the server response body. A progress event only describes transfer progress; it does not mean the server accepted or stored the file.

## 6. Keep transfer progress and completion separate

A useful UI state distinguishes an in-progress transfer from a completed response:

~~~ts
type UploadState =
  | { kind: 'uploading'; loaded: number; total?: number; percent: number | null }
  | { kind: 'complete'; result: UploadResult };
~~~

For each event, handle UploadProgress and Response separately. If the server responds with an error, HttpClient emits through the Observable error channel; it is not represented as a successful progress event.

## 7. Download progress

For a binary file, request a Blob and enable download progress:

~~~ts
this.http.get('/api/files/42/content', {
  observe: 'events',
  responseType: 'blob',
  reportDownloadProgress: true
});
~~~

The DownloadProgress events can update a meter. When the final Response event arrives, its body is the completed Blob.

With text responses, a download progress event may include partialText. Do not treat partial text as a complete document; it is an intermediate chunk.

## 8. Progress percentages are estimates

A determinate percentage is possible only when total is available:

~~~ts
const percent = event.total
  ? Math.round((event.loaded / event.total) * 100)
  : null;
~~~

Even when a total is available, progress reaching 100 percent means the bytes were transferred, not necessarily that the server finished processing the request. Wait for the final response before showing the operation as successful.

## 9. Cancellation and errors

Unsubscribing from an in-flight HttpClient request can abort the client-side operation. It does not guarantee that a server that already received an upload has deleted the data or reversed its work. Use an explicit server cancellation or cleanup API if the product requires that guarantee.

Network failures, timeouts, and backend errors arrive through the Observable error channel. The UI should clear or mark progress as failed and allow an appropriate retry. Detailed retry and error policies are handled in a later lesson.

## 10. Performance and accessibility

- Enable progress only for operations whose UI consumes it.
- Progress events can arrive frequently; avoid expensive work for each event.
- Show a percentage only when total is known.
- Provide an indeterminate state when total is unavailable.
- Include an accessible text label such as “Uploading report: 42 percent”.
- Announce meaningful state changes rather than every byte update.
- Do not show success until the final response has completed successfully.

## Common mistakes

### Enabling upload progress with Fetch

Fetch does not report upload progress through HttpClient. Use withXhr for the browser upload use case.

### Forgetting observe: 'events'

Progress flags alone do not make the consumer receive the event stream. Set observe to 'events'.

### Assuming total always exists

The total may be missing. Treat it as optional and use indeterminate progress when necessary.

### Treating a progress event as success

Only the final Response indicates the server returned a completed response. Handle Observable errors separately.

### Forcing a multipart Content-Type

Let the browser build the FormData Content-Type with its boundary.

### Enabling deprecated reportProgress in new code

Use reportUploadProgress and reportDownloadProgress where available. Use the version-specific API for older Angular releases.

## Key idea

Progress is part of the HttpClient event stream. Enable it only when needed, check the event type, treat total as optional, and wait for the final response before declaring success. Upload progress specifically requires an XHR backend in current Angular.

## Related references

- Q&A: qnas/088-upload-and-download-progress-events.md
- Examples: examples/088-upload-and-download-progress-events.md
- Angular making HTTP requests: https://angular.dev/guide/http/making-requests
- HttpEventType: https://angular.dev/api/common/http/HttpEventType
- Upload progress event: https://angular.dev/api/common/http/HttpUploadProgressEvent
- Download progress event: https://angular.dev/api/common/http/HttpDownloadProgressEvent
- withXhr: https://angular.dev/api/common/http/withXhr
