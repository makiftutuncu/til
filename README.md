# til
A 'Today I Learned' notebook

## 7 February 2018, Wednesday
`BlistServlet` doesn’t support overloading for methods with special names (create, index etc.).

**Problem:** I have a REST endpoint to list some items. It is calling some internal service and that service is doing some external calls. At the end of the day, the result of those calls get cached and eventually I get some incorrect (potentially dangerous) data from my endpoint. To disable caching, I would need to modify HTTP headers of the external call the service makes.

**Question:** How do I do that with minimum modifications and without breaking anything?

**Solution:** Create an overloaded version of the service method to add a `disableCache` parameter. Before making the external request, modify the request headers if the flag is set to `true`. Update all calls to the service method and pass `false` to `disableCache` parameter (because we don’t want existing behaviour to change, hence we don’t disable the cache by default). Only pass it `true` from your REST endpoint (or anywhere else where you want to disable the cache).
