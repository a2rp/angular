# Upload and Download Progress Events - Q&A

## Events and configuration

### 1. How do I receive progress events from HttpClient?

Use observe: 'events' and enable the direction the UI needs with reportUploadProgress or reportDownloadProgress. Progress reporting is disabled by default because events can be frequent and have a performance cost.

### 2. What is the difference between reportUploadProgress and reportDownloadProgress?

reportUploadProgress requests upload progress events for the request body. reportDownloadProgress requests download progress events for the response body. Enable only the direction your feature uses.

### 3. What happened to reportProgress?

Current Angular documentation deprecates reportProgress in favor of separate reportUploadProgress and reportDownloadProgress options. Older Angular versions may use reportProgress: true. Use the API supported by the project's Angular version.

### 4. Why do I need observe: 'events'?

The default HttpClient Observable emits the response body. observe: 'events' returns the lower-level event stream, which includes lifecycle and progress events.

### 5. What event types can appear in the stream?

Common event types are Sent, UploadProgress, ResponseHeader, DownloadProgress, Response, and User. Check the event type before reading progress-specific or response-specific properties.

### 6. How do I know the transfer finished successfully?

Wait for HttpEventType.Response. A progress event only reports transfer state; it does not confirm that the backend accepted, processed, or stored the data.

### 7. How do I type the final response in an event stream?

Use the request's response type and narrow the event to HttpResponse<T> by checking event.type against HttpEventType.Response. The HttpResponse body may be null.

## Uploads

### 8. Why does upload progress throw with the default backend?

The default Fetch backend does not support upload progress. In current Angular, enabling reportUploadProgress with Fetch throws an error. Configure HttpClient with withXhr() when the browser upload flow needs upload progress.

### 9. Does withXhr need to be enabled for download progress too?

The documented limitation is specifically upload progress. Fetch can report download progress. Choose a backend based on the operation and the Angular version's documented capabilities.

### 10. Does withXhr work for server-side rendering?

Angular recommends the default Fetch backend for SSR and warns against using XHR there. If an application needs XHR for browser upload progress, consider how server-side rendering is configured and avoid relying on XHR as the server transport.

### 11. How do I send a file with upload progress?

Put the file in FormData, send the FormData as the request body, set observe: 'events', and enable reportUploadProgress. Configure XHR as needed for upload progress.

### 12. Should I manually set Content-Type to multipart/form-data?

No. Let the browser set the multipart boundary in the Content-Type header when sending FormData. Manually setting only multipart/form-data can omit the boundary the server needs to parse the body.

### 13. Does upload progress reaching 100 percent mean the upload succeeded?

No. It means the request body transfer completed. The backend can still reject or fail to process it. Wait for the final response and handle errors separately.

## Downloads and progress values

### 14. What do loaded and total mean?

loaded is the number of bytes transferred so far. total is the expected total byte count when it is available. Both upload and download progress event types expose these fields.

### 15. Can total be undefined?

Yes. The size may be unknown or not computable. Show an indeterminate progress indicator instead of calculating a percentage when total is absent or zero.

### 16. What formula should I use for a percentage?

When total is a positive number:

~~~ts
const percent = Math.round((event.loaded / event.total) * 100);
~~~

Guard for missing or zero total before dividing.

### 17. Can a download progress event contain part of the response body?

For text responseType, HttpDownloadProgressEvent may expose partialText. It is an intermediate portion of a response, not the complete document. The final response arrives as HttpEventType.Response.

### 18. How do I download a binary file while tracking progress?

Request a Blob or ArrayBuffer response, set observe: 'events', and enable reportDownloadProgress. When the final Response event arrives, read the completed binary body.

## Lifecycle, performance, and errors

### 19. Why should I not enable progress for every request?

Progress events can arrive frequently and add runtime work. Enable them only for operations where the interface consumes the progress information.

### 20. Does unsubscribing cancel an upload?

Unsubscribing can abort the in-flight client request. It cannot guarantee that a server which already received the bytes has removed the uploaded data or reversed processing. Use a server-side cancellation or cleanup endpoint if that guarantee is required.

### 21. How do network and server failures appear?

Failures are emitted through the Observable error channel, not as a successful Response event. Clear or mark the UI as failed, then let the user retry when appropriate.

### 22. Should progress updates be announced to screen readers for every event?

Usually no. Frequent byte updates can overwhelm assistive technology. Use a progress element with an accessible label and announce meaningful changes, such as the beginning, completion, or failure, at a suitable frequency.

### 23. Does a completed byte transfer mean the server finished its work?

Not necessarily. Transfer completion and server-side processing are different stages. Show success only after the final successful response or an application-specific completion signal.

### 24. Why is observe: 'events' missing the expected TypeScript type?

The observe value selects an HttpClient overload. Keep it as a literal value, including when options are extracted into a variable:

~~~ts
const options = {
  observe: 'events' as const,
  reportDownloadProgress: true
};
~~~

## Quick reference

| Goal | Required setup |
| --- | --- |
| Upload progress | observe: 'events', reportUploadProgress: true, and XHR backend |
| Download progress | observe: 'events', reportDownloadProgress: true |
| Percentage display | Use loaded and total only when total is available and positive |
| Final success | Handle HttpEventType.Response |
| Send a browser file | FormData; let the browser set multipart Content-Type |
| Older Angular compatibility | Use the progress option supported by the installed version |

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="087-url-parameters-headers-response-types-and-credentials.md">087 - URL Parameters, Headers, Response Types, and Credentials</a></td><td><a href="../lessons/088-upload-and-download-progress-events.md">Lesson</a></td><td><a href="../examples/088-upload-and-download-progress-events.md">Example</a></td><td><a href="089-functional-interceptors-and-request-context.md">089 - Functional Interceptors and Request Context</a></td>
</tr>
  </tbody>
</table>
