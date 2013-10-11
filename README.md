deferredApi.js
==============

Interact with iframes, web workers, local objects, or a web server using identical deferred interfaces.

Basic Usage
===========

Easily create deferred wrappers for objects hosted on:

* a webserver (standard AJAX)
```javascript
  var service = deferredApi.ajax( url, methodList )
```

* web workers
```javascript
  /*** workerScript.js ***/
  importScripts('deferredApi.js');
  
  var service = {                     // create service
      foo: function() { return 3; },
      bar: function() { return 7; }
  };
  deferredApi.host(service, portId);
```
```javascript
  /*** main application script ***/
  
  function getService() {
      // we can specify the API we are expecting and it will be mocked-in
      // otherwise we have to wait for the service to resolve before any of its methods exist
      var service = deferredApi.worker('workerScript.js', 'foo bar');
      service.foo().then(function(val) {
          console.log(val); // 3
      });
      
      // fall-back to Ajax if the users browser doesn't support workers, same interface
      return service;
  }
```

* iframes (including cross-domain)
```javascript
```

* the local JavaScript context
```javascript
```

Simple AJAX Wrapper
===================

Consider a simple JSON web service and a jQuery client wrapper to encapsulate the web service details.

    var webService = {
      baseUrl: 'https://someservice.com/myService',
      add3: function(arg0) {
        return $.getJSON(this.baseUrl + '/add3', { args: $.makeArray(arguments) });
      },
      mult5: function(arg0) {
        return $.getJSON(this.baseUrl + '/mult5', { args: $.makeArray(arguments) });
      }
    };

The web service is trivial for demonstration purposes, but you can imagine a service that provides results from
querying your server-side database instead.  JavaScript code on your web page can now interact with this
service using the $.Deferred type interface provided by the return value of $.getJSON().

    var x, y;
    var s = webService;
    
    s.add3( 5 )
      .then(function( xVal ) {
      x = xVal;
      return s.mult5( x );
    }).then(function( yVal ) {
      y = yVal;
    });

Non-Ajax Deferred Interfaces
============================

Because we have encapsulated the service into methods that each present a $.Deferred interface client code no longer
needs to know where or how those methods are implemented as long as they comply with that interface.  For example,
during the development process we could replace the above remote service with a local mockup implementation.

    var mockImplementation = {
      add3: function(arg0) { return arg0 + 3; },
      mult5: function(arg0) { return arg0 * 5; }
    };
    
    var mockService = deferredApi.local( mockImplementation );

Here we have used deferredApi.local() to "downgrade" a synchronous implementation so that it presents the same
asynchronous interface as the remote Ajax implementation will once the application enters production.  Code written
against this mock interface during development will work seemlessly with a remote Ajax implementation later on.

Other "Hosts"
=============

There are a number of reasons to host functionality outside of the web page loaded in the users browser.  Standard Ajax
services give your application access to a data, security and CPU context that aren't available directly from within
the clients JavaScript context.  Fetching data on demand and off-loading slow running operations improves the user
experience by lowering initial page-load times and not blocking the browsers UI thread.

Recent developments in browser technology allow us to capture some of these benefits in other ways:

* Rather than reloading data from the server every time a page is reloaded we can avoid round-trips by caching data in localStorage.
* Similarly Web Workers allow us to offload long-running operations to a local background thread instead of the server.
* Loading and running code within an iframe can shift the security context for JavaScript running in a separate domain
via a 3rd party tag or bookmarklet.

Communicating with both web workers and iframes, like Ajax, is asynchronous.  Using localStorage as a cache for remote
data should also be wrapped in an asynchronous interface:

    var proxyFnNames = Object.getOwnPropertyNames(Object.getPrototypeOf(localStorage));
    var wrappedStorage = deferredApi.local(localStorage, proxyFnNames);

deferredApi.js enables the creation of a standard deferred interface for code that is hosted within any of these contexts,
along with facilities to wrap a deferred interface when none exists.  Programming against a deferredApi enables you to
relocate services down the road without those changes impacting your application code.










```javascript
    !!service['foo'];           // false, the api hasn't loaded yet
    
    // the API for the service will be resolved asynchronously but loaded in place
    service.then(function(result) {
        service === result;     // true, the service resolves to itself after the api is loaded
        !!service['foo'];       // true, the api was loaded in place on the service object
    });
    
    // or we can specify the api we are expecting when we request the service
    // any calls will be buffered until the service resolves, then forwarded
    var service = deferredApi.worker('workerScripts.js', 'foo bar');
    service.foo().then(function(val) {
        val; // 3, after the service is resolved, the synchronous call to 'foo' is forwarded
    });
```
