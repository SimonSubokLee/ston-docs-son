.. _caching-policy:

Chapter 4. Caching Policy
*************************

This chapter will cover the Time To Live (TTL), the Caching Key, and the expiration policy, which are fundamental to the service. Stored content is only available for the amount of time given by the TTL. Standard HTTP protocol specifies that Cache-Control can be used to set the TTL. Service quality can be improved through the use of various TTL policies and :ref:`caching-purge`.

HTTP has various standards to classify content. As such, various Caching Keys exist as well. Not only will there be less load on the origin server with less content changes, but the service will be easier to scale up as well. In this chapter, we will discuss various ways to set up optimized expiration policies for a service.

If you want to apply the upcoming configurations to all virtual hosts as a default configuration, you can do so under the ``<VHostDefault>`` To do the opposite and apply them to specific virtual hosts, use the ``<Vhost>`` tag.

The **Caching-Key** is a distinct value that classifies content. It is similar to how a file system classifies files using a distinct path (e.g. ./user/conf.txt). It is easy to confuse Caching Keys with URLs. However, depending on the various functions of HTTP, identical URLs could return different content.


.. toctree::
   :maxdepth: 2



.. _caching-policy-ttl:

Time To Live (TTL)
====================================

The TTL is the amount of time stored content stays available. A longer TTL setting will reduce load on the origin server, but modifications will take longer to be applied because they must wait until the TTL expires. Conversely, a shorter TTL will mean higher load on the origin server due to more frequent requests for modification checks. The service can run smoothly when an appropriate TTL setting is found and the origin server load is decreased. When a TTL is set, it will not change until it expires. A new TTL is only applied to a file when the old TTL expires. Administrators can use API functions such as :ref:`api-cmd-purge`, :ref:`api-cmd-expire`, :ref:`api-cmd-expireafter`, and :ref:`api-cmd-hardpurge` to change the TTL.


Default TTL
---------------------

By default, the TTL is set based on the response of the origin server. The stored content will be provided until the TTL expires. When the TTL expires, a check request for modified content (**If-Modified-Since** or **If-None-Match**) will be sent to the origin server. If the origin server returns a 304 Not Modified response, the TTL is extended. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTL>
        <Res2xx Ratio="20" Max="86400">1800</Res2xx>
        <NoCache Ratio="0" Max="5" MaxAge="0">5</NoCache>
        <Res3xx>300</Res3xx>
        <Res4xx>30</Res4xx>
        <Res5xx>30</Res5xx>
        <ConnectTimeout>3</ConnectTimeout>
        <ReceiveTimeout>3</ReceiveTimeout>
        <OriginBusy>3</OriginBusy>
    </TTL>    
    
Except for ``Ratio`` (0~100), all units are in seconds.

-  ``<Res2xx> (default: 1800 sec, Ratio: 20, Max: 86400)``
   Sets the TTL when the origin server responds with 200 OK. When content is first stored, it is set to expire after ``<Res2xx>`` seconds. After the TTL expires, if the content on the origin server has not changed (304 Not Modified), then the TTL is extended according to the ``Ratio`` value (0~100). The TTL can be increased up to ``Max`` seconds.

-  ``<NoCache> (default: 5 sec, Ratio: 0, Max: 5, MaxAge: 0)``
   This function works identically to ``<Res2xx>``, but is only used when the origin server responds with "no-cache". ::
   
      cache-control: no-cache or private or must-revalidate
    
   If ``MaxAge`` is greater than 0, max-age can be applied.
    
   .. figure:: img/nocache_maxage.png
      :align: center

      Files are cached for Max-Age seconds.

-  ``<Res3xx> (default: 300 sec)``
   Sets the TTL when the origin server responds with "3xx". This function is frequently used for redirects.

-  ``<Res4xx> (default: 30 sec)``
   Sets the TTL when the origin server responds with "4xx". Responses are often **404 Not Found**.

-  ``<Res5xx> (default: 30 sec)``
   Sets the TTL when the origin server responds with "5xx". This usually occurs due to an internal error in the origin server.

-  ``<ConnectTimeout> (default: 3 sec)``
   Sets the TTL when the origin server cannot be reached. If the content is already saved, then the TTL is extended for ``<ConnectTimeout>`` seconds. If the content is not saved, then an error status will be returned for ``<ConnectTimeout>`` seconds. The intention is to lessen the burden on the origin server for a TTL amount of time, not to provide an error status to the service.

-  ``<ReceiveTimeout> (default: 3 sec)``
   Sets the TTL when the connection is successful but data could not be acquired. The intention is the same as ``<ConnectTimeout>``.

-  ``<OriginBusy> (default: 3 sec)``
   If the :ref:`origin-busysessioncount` condition is satisfied, then the TTL of expired content will be extended for the set amount of time without making requests to the origin server. This is so that additional load is not placed on the origin server.
   
.. note::

   If the TTL is set to zero, content will expire as soon as it is provided. If you want to have the origin server respond to all requests, a bypass is recommended.
   

.. _caching-policy-customttl:
   
Custom TTL
---------------------

Separate TTLs can be set for each URL. Fixed TTLs can be set for content that match specific URLs or patterned URLs. This can be configured in /svc/{virtual host name}/ttl.txt. ::

    # /svc/www.example.com/ttl.txt
    # Commas (,) are used as delimiters and the unit of time is seconds.
    
    *.jsp, 10
    /,5
    /index.html, 5
    /script/*.js, 300
    /image/ad.jpg, 1800
    

Even if you add \*.html to set separate TTLs for all pages (e.g. html, php, jsp), this will not set the TTL for the first page (/). The HTTP protocol cannot identify what page is set as the first page (e.g. index.php, default.jsp) for the origin server. Therefore, in order to set a TTL for every page, a "/" should always be added.
    
    
TTL Priority
---------------------

The order in which TTL is applied can be configured. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTL Priority="cc_nocache, custom, cc_maxage, rescode">
        ... (omitted) ...
    </TTL>    

This can be configured using the ``Priority (default: cc_nocache, custom, cc_maxage, rescode)`` item in the ``<TTL>`` tag.

- ``cc_nocache`` When the origin server responds with "Cache-Control: no-cache".
- ``custom`` :ref:`caching-policy-customttl`.
- ``cc_maxage`` When the origin server displays maxage in Cache-Control.
- ``rescode`` The default TTL for response codes from the origin server.


Abnormal TTL Extension
----------------------

It's obvious that an error has occurred if there is no response from the origin server, but there are cases when an error will have occurred while the server continues to respond normally at times. For example, it may lose connection with the storage that has the content, or it may decide that regular service is unavailable. The response will usually be a 4xx response (usually **404 Not Found**) for the former or a 5xx response (usually **500 Internal Error**) for the latter.

However, if the content is already stored, then it is more effective to extend the TTL to prevent total service failure rather than rely the origin server's responses. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTLExtensionBy4xx>OFF</TTLExtensionBy4xx>
    <TTLExtensionBy5xx>ON</TTLExtensionBy5xx>

-  ``<TTLExtensionBy4xx>``

   -  ``OFF (default)`` Updates content with a 4xx response.
   
   -  ``ON`` Acts as if the response was a **304 Not Modified**.
   
You should also check to see if the 4xx response was intentional.
       
-  ``<TTLExtensionBy5xx>``

   -  ``ON (default)`` Acts as if the response was a **304 Not Modified**.
   
   -  ``OFF`` Updates content with a 5xx response.

A normal server will not return a 5xx response, as it is used to reduce the load on the origin server by invalidating content from a temporary server error.
    

.. _caching-policy-renew:

Update Policy
====================================

Content will be updated after the TTL expires and the update check is confirmed at the origin server.

   .. figure:: img/perf_refreshexpired.jpg
      :align: center
      
      A response after checking for modifications.

1. The TTL is valid and an immediate response is given.

#. The TTL has expired, so a modification check (If-Modified-Since) is requested to the origin server. There is no response to the client until this check is performed.
   
#. When a response is returned by the origin server, either the TTL is extended or the content is swapped. With confirmation from the origin server, a response will be made to the client.
   
#. An immediate response is given for the checked content until its TTL expires.

For services like HD videos or games where transfer speed is more important than the response rate, this process is not very meaningful. With bulk data, it doesn't matter that the origin server can respond within ten seconds because the transfer time takes much longer. Rather, since it is content that isn't accessed frequently, it will need more renewal checks.

Meanwhile, online shopping malls are a different story. Web pages loading quickly is more important than anything else. The client's screen must load in 1~2 seconds. In other words, the transfer time is more important than the response rate.

At this point, if the TTL expires and a update check must be performed, it could cause a huge delay. Considering the fact that most shopping malls must handle millions of items of content simultaneously, you must assume that update checks are constantly occurring on the origin server. 

What we want is to stably transfer cached content regardless of any origin server error or delay.

   .. figure:: img/perf_refreshexpired2.jpg
      :align: center
      
      We're not afraid of errors!
      
These different requirements have led to the development of the background content renewal function. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <RefreshExpired>ON</RefreshExpired>    

-  ``<RefreshExpired>``

   -  ``ON (default)`` Responds after the modification check.
   
   -  ``OFF`` Responds without waiting for the modification check response. Content will be swapped when new content is downloaded.

``OFF`` The OFF setting is generally used because content is not changed very often.

   .. figure:: img/perf_refreshexpired5.jpg
      :align: center
      
      There is no need to wait if content is not sensitive to changes.

As seen in the above image, because updating from the origin server takes place in the background, the cached content can be immediately sent to the client without waiting. If the origin server responds with **304 Not Modified**, the TTL is extended. When the file is updated and the origin server responds with **200 OK**, the file is smoothly replaced after it is fully downloaded. After the file is updated, users will receive the new file (colored yellow). Regardless of variables such as network failures or server failures, content updating will take place in the background and result in no service delays.


TTL Expiration when Clients Request no-cache
--------------------------------------------

If there is at least one no-cache setting in the client's HTTP request, then content can be made to expire right away. ::

    GET /logo.jpg HTTP/1.1
    ...
    cache-control: no-cache or cache-control:max-age=0
    pragma: no-cache
    ...
    
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <NoCacheRequestExpire>OFF</NoCacheRequestExpire>    

-  ``<NoCacheRequestExpire>``

   -  ``OFF (default)`` The request is ignored.
   
   -  ``ON`` The TTL is made to expire right away.
   
The expired content follows the :ref:`caching-policy-renew`.


.. _caching-policy-accept-encoding:

Accept-Encoding Header
====================================

Even though HTTP requests may be for the same URL, depending on the existence of an Accept-Encoding header, different content may be cached. When STON sends a request to the origin server, it has no idea of knowing if the file is compressed or not, and it can't check for compression every time, either.

   .. figure:: img/acceptencoding.png
      :align: center

      It's impossible to know what kind of response the origin server will give.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <AcceptEncoding>ON</AcceptEncoding>    

-  ``<AcceptEncoding>``

   -  ``ON (default)`` Will recognize the Accept-Encoding header sent by the HTTP client.
   
   -  ``OFF`` Will ignore the Accept-Encoding header sent by the HTTP client.
    
If the origin server does not support compression, or if the bulk file does not require compression, then it is recommended to set ``<AcceptEncoding>`` to ``OFF``.

.. _caching-policy-casesensitive:

Case Sensitivity
====================================

STON is unable to tell on its own if the origin server can differentiate between upper and lower case letters.

   .. figure:: img/casesensitive.png
      :align: center

      Either the content is the same or a 404 will occur.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <CaseSensitive>ON</CaseSensitive>    

-  ``<CaseSensitive>``

   -  ``ON (default)`` Differentiates between upper and lower case letters.
   
   -  ``OFF`` Does not differentiate. All letters are processed as lower case.

    
.. _caching-policy-applyquerystring:
    
QueryString Differentiation
====================================

It is not necessary to identify a query string unless the content is dynamically created by the query string If a URL contains a meaningless random value or a constantly changing time value, then it can create a lot of load on the origin server.

   .. figure:: img/querystring.png
      :align: center

      If the content is not dynamic, it is more likely to be identical.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <ApplyQueryString Collective="OFF">ON</ApplyQueryString>

-  ``<ApplyQueryString>``

   -  ``ON (default)`` Identifies query strings. If one of the exception cases is met, the query string is ignored.
   
   -  ``OFF`` Ignores query strings. If one of the exception cases is met, the query string is identified.
    
Query string exception cases are saved at /svc/{virtual host name}/querystring.txt. ::

    # ./svc/www.example.com/querystring.txt
    
    /private/personal.jsp?login=ok*
    /image/ad.jpg

Note that the exception case changes in meaning depending on the setting of ``<ApplyQueryString>``. Specific or patterned URLs (only \* patterns are allowed) can be used in the configuration.

The ``Collective`` property comes into play when the :ref:`caching-purge` API is called.

-  ``Collective``

   -  ``OFF (default)`` Only the URL parameter will be targeted.
   
   -  ``ON`` All content with URLs containing query strings will be targeted, not just the URL parameter.
   
If the ``Collective`` property is set to ON and there are many files, CPU load will become higher. It may take longer to search for the correct files, and unforeseen problems may occur. It is recommended to call the :ref:`caching-purge` API using clearly defined URLs with query strings as much as possible.




Vary Header
====================================

Content can be classified through the use of Vary headers. Generally, Vary headers are the primary cause of sudden drops in performance on the cache server. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader />    
    
-  ``<VaryHeader>``

   Configures the list of Vary headers to be supported among the ones returned by the origin server. Commas (,) are used as delimiters.

For example, if the origin server returns the following as the Vary header, it will be ignored because it is not set in <VaryHeader>. ::

    Vary: Accept-Encoding, Accept, User-Agent

To exclude User-Agent and only recognize Accept-Encoding and Accept headers, do the following. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>Accept-Encoding, Accept</VaryHeader>    
    
To recognize all Vary headers sent by the origin server, do the following. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>*</VaryHeader>    


.. _caching-policy-post-method-caching:

POST Request Caching
====================================

POST requests can be configured so that they are cached. POST requests have the same characteristics as URLs, but may differ in Body data. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <PostRequest MaxContentLength="102400" BodySensitive="ON">OFF</PostRequest>    

-  ``<PostRequest>``

   -  ``OFF (default)`` If a POST request arrives, the session ends.
   
   -  ``ON`` POST requests are cached.
   
Most POST request processing cases use the Body data as Caching Keys. Detailed configuration can be made using the ``BodySensitive`` property and exception cases.

-  ``BodySensitive``

    -  ``ON (default)`` Body data is recognized as a Caching Key. Maximum length is set by the ``MaxContentLength (default: 102400 bytes)`` property. If one of the exception cases is met, the Body data is ignored.
    
    -  ``OFF`` Body data is ignored. If one of the exception cases is met, the Body data is recognized.
   
POST request exceptions can be set in the file /svc/{virtual host name}/postbody.txt. ::
    
    # /svc/www.example.com/postbody.txt
    
    /bigsale/*.php?nocache=*
    /goods/search.php
    
Note that the exception case changes in meaning depending on the setting of ``BodySensitive``. Specific or patterned URLs (only \* patterns are allowed) can be used in the configuration.

It is possible to mix up this setting with :ref:`bypass-getpost`. POST requests may not be cached at all depending on the ``<BypassPostRequest> (default: ON)`` setting. As such, to cache POST requests, either ``<BypassPostRequest>`` must be set to ``OFF`` or an exception case must be set. In order of priority:
        
* Bypasses the origin server if bypass conditions (:ref:`bypass-getpost`) are met.
* Terminates the connection if there is no Content-Length header.
* Caches files if ``PostRequest`` is set to ``ON`` and Content-Length does not exceed ``MaxContentLength``.
* Terminates the request if none of the above scenarios are encountered.

.. note::

    If ``MaxContentLength`` is set to too high of a value, a lot of memory will be needed to manage the Caching Key. It is best to set it as small as possible.
