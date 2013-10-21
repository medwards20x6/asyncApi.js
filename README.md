asyncApi.js
==============

Interact with iframes, web workers, local objects, or a web server using identical deferred interfaces.

Basic Usage
===========

Consider a vanilla object implementation of a simple service:

```javascript
var service = {
    add3: function( a ) { return a + 3; }
};
```

Now imagine that during development the service implementation changes in a way that requires is to be relocated
to the server and accessed via an AJAX call.  All of your code that uses the service must now be refactored to
access it asynchronously.

```javascript
// Synchronous
var x = service.add3( 5 );
// code that uses x

// Using a jQuery.Deferred() style AJAX interface becomes:
var x;
service.add3( 5 ).then(function( xVal ) {
    x = xVal;
    // code that uses x
});
```

You can use asyncApi.js to create wrappers that always provide a uniform asynchronous service interface.  The service
may be hosted locally, on the server, lazily loaded as an [AMD][1], or hosted within another browser window, tab, iframe
or [web worker][3].  A uniform asynchronous interface shields client code from changes to the service implementation,
initialization time or hosting location.

```javascript
// service client
var localAsyncService = asyncApi.local(service), // synchronous service wrapped as async
    ajaxService       = asyncApi.ajax(rootUrl),
    browserService    = asyncApi.connect(iframeOtherWindowOrWorker), // existing service
    lazyService       = asyncApi.loadAMD(amdUrl),
    onDemandWorker    = asyncApi.loadWorker(workerScriptUrl),
    onDemandFrame     = asyncApi.loadFrame(iframeUrl);

// within a service host (window or worker)
var service = implementation();
asyncApi.host( service );
```

asyncApi.js is built on basic serialization, deserialization and point-to-point message routing and response between an
async interface and its implementation across windows and workers.  Multiple service implementations can be hosted
within a single window or worker by using portIdentifiers and messages are multi-plexed then routed via
window.postMessage() and worker.postMessage().  Services can be configured to support multiple connections.

Why?
====

There are several reasons to host a service outside of the browsers main UI thread:

* Accessing data that is unavailable within the browser context
* Running in a separate thread to avoid locking the user interface
* Accessing data and performing computations in a safe, clean security context

These should all be familiar to anyone using AJAX, but some of these benefits can also be provided locally
by modern web standards using [IndexedDB][2], [web workers][3], and/or [cross-frame messaging][4].  Hiding service
implementations behind a standard interface enables using the same client code with [progressively enhanced][5]
service implementations for modern browsers that can increase UI performance and reduce server load.

For example: a data access API may use AJAX by default.  If the browser supports IndexedDB data can be cached and
retrieved locally.  CPU intensive operations that are part of the API can be run locally using web workers if they
are supported or off-loaded to the server via AJAX if not.  All of these implementations support the same asynchronous
API:

```javascript
var dataService = initDataService();
dataService.expensiveOperation(arg0, arg1).then(function( result ) {
    // process the result
});
```

The syntax supports any combination of implementations listed above.  The client code doesn't care where the data comes
from or where the processing occurs.

[1]: http://en.wikipedia.org/wiki/Asynchronous_module_definition
[2]: https://developer.mozilla.org/en-US/docs/IndexedDB
[3]: https://developer.mozilla.org/en-US/docs/Web/Guide/Performance/Using_web_workers
[4]: https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage
[5]: http://en.wikipedia.org/wiki/Progressive_enhancement
