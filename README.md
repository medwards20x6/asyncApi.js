deferredApi.js
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

Now imagine that during development the service implementation changes in a way that it needs to be relocated
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

deferredApi.js creates asynchronous wrappers that provide a uniform service interface.  The service can be
hosted locally, on the server, lazily loaded as an [AMD][1], or hosted within another browser window, tab, iframe
or web worker.  A uniform asynchronous interface shields client code from changes to the service implementation,
initialization time or hosting location.

```javascript
// service client
var localAsyncService = deferredApi.local(service), // synchronous service wrapped as async
    ajaxService       = deferredApi.ajax(rootUrl),
    lazyService       = deferredApi.loadAMD(amdUrl),
    browserService    = deferredApi.connect(frameTabWindowOrWorker), // existing service
    onDemandWorker    = deferredApi.loadWorker(workerScriptUrl),
    onDemandFrame     = deferredApi.loadFrame(iframeUrl);

// within a service host (window or worker)
var service = implementation();
deferredApi.host( service );
```

Why?
====

There are several reasons to host a service outside of the browsers main UI thread:

* Accessing data that is unavailable within the browser context
* Running in a separate thread to avoid locking the user interface
* Accessing data and performing computations in a safe, clean security context

These should all be familiar to anyone using AJAX, but some of these benefits can also be provided locally
by modern web standards using [IndexedDB][2], [web workers][3], and/or [cross-frame messaging][4].  For example,
the original motivation for the library:

An application from example.com is loaded via a bookmarklet or browser extension into a page hosted on a 3rd party
domain.  The application needs to load data and run complex potentially time consuming operations over the data.  The
example.com application is written to access all data through an asynchronous interface.

The asynchronous interface behaves differently depending on the features supported by the browser.  On a legacy browser
data requests are resolved by issuing cross-domain JSONP requests to the server.  On a modern browser, a cross-domain
iframe is initialized that spawns web worker processes capable of loading cached data out of IndexedDB storage for
the example.com domain.  AJAX requests are only issued for data versioning updates and non-cached data.

The interfaces between the main page and the iframe, the iframe and the workers, and the workers and the data layer can
each be hidden behind an async interface.  If the browser doesn't support the features further down the stack the
implementation can fall-back to issuing AJAX requests to the server.

Transparently handling requests locally or remotely depending on browser features can increase performance from the
users perspective while decreasing load on the servers.  As more and more users upgrade to modern browsers over time
the benefits are amplified.  Routing requests through standard async interfaces makes developing fall-backs
straighforward.  

[1]: http://en.wikipedia.org/wiki/Asynchronous_module_definition
[2]: https://developer.mozilla.org/en-US/docs/IndexedDB
[3]: https://developer.mozilla.org/en-US/docs/Web/Guide/Performance/Using_web_workers
[4]: https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage
