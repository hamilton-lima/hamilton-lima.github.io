---
title: 'CORS is your friend'
date: Wed, 19 Oct 2016 23:36:58 +0000
draft: false
tags: ['beepify.io', 'CORS', 'playframework', 'selfhackaton', 'server', 'test']
---

As soon we started to test the integration of the server side with the mobile application we found out that we were missing one little detail

> Enable Cross Origin Resource Sharing!

Those beatiful words means: We sould tell the server to allow other servers to call our services :) As we are not doing any kind of authentication we should let any server make the calls. So we just went to [http://enable-cors.org/server.html](http://enable-cors.org/server.html) waiting to find some answers, sadly there were no playframework information there, but as this would be implementede easily as a Java Filter (we should only add one line to the response headers) we found the proper documentation at [https://www.playframework.com/documentation/2.5.x/CorsFilter](https://www.playframework.com/documentation/2.5.x/CorsFilter) Of course we could implement a [filter](https://www.playframework.com/documentation/2.5.x/ScalaHttpFilters) to do it, but is much simpler to just use what is already tested :) so we just follow the instructions that are in summary:

*   enable the filter at build.sbt, libraryDependencies += filters
*   make the changes to the Filter class, to use the CORSFilter
*   configure the parameters of the CORS filter at applications.conf

```
play.filters {
 cors {
 # allow all paths
 pathPrefixes = \["/"\]
 
 # allow all origins (You can specify if you want)
 **allowedOrigins = null**
 
 allowedHttpMethods = \["GET", "POST", "PUT", "DELETE"\]
 
 # allow all headers
 allowedHttpHeaders = null
 }
 ...
}
```

Now all your attention to this:

> allowedOrigins should be NULL to accept ANY origin.

Other warning before you loose some time trying to find out why the "Access-Control-Allow-Origin: \*" header is NOT showing at all in the response, like this screenshot below from the [Chrome inspector](https://developer.chrome.com/devtools). ![no_cors_response_headers](/images/2016/10/no_cors_response_headers.png)

> ...by default no headers are exposed...

So don´t expect the header to show unless you setup to show. To test if the CORS is working properly use this site that offer an quick and easy test environment that make a call to your service to check if CORS is ok. [http://www.test-cors.org/](http://www.test-cors.org/)