.. _caching-policy:

Chapter 4. Caching Policy
******************

This chapter will explain the TTL(Time To Live), Caching-Key and expiration policy, which are fundamental to the service.
Stored content is only available while the TTL is valid.
Standard HTTP protocol specifies Cache-Control for setting the TTL.
However, this is not mandatory. 
Various TTL policies and :ref:`caching-purge` will improve service quality.

HTTP has various standards that classify content. 
There could be various Caching-Keys as well. 
Updating content less frequently not only helps reduce the load on the origin server, but also makes it easier to scale up the service.
In this chapter, several methods to establish an optimized expiration policy for a service will be discussed.

In order to use the upcoming configuration as a default setting for all virtual hosts, place it under the ``<VHostDefault>``.
On the contrary. To use the configuration for the specific virtual host, place it under the <Vhost> tag.

The **Caching-Key** is a unique value that distinguishes contents. 
In a similar way, a file system has a unique path for a file (e.g. /usr/conf.txt).
Occasionally people confuse the Caching-Key with URL.
However, depending on various functions of HTTP, identical URLs could return different content.


.. toctree::
   :maxdepth: 2



TTL (Time To Live)
====================================

TTL stands for the expiration time of saved contents.
Having a longer TTL setting reduces the burden on the origin server, but modifications will be applied after TTL expires.
On the contrary, a shorter TTL setting will increase the load on the origin server due to frequent requests for modification checks.
The beauty of TTL management is that it finds an appropriate setting for reducing the load on origin server.
Once the TTL is set, it will not change until the TTL is expired.
The new TTL will be applied when the old TTL for the file is expired.
Administrator can change TTL setting by using these APIs, :ref:`api-cmd-purge` , :ref:`api-cmd-expire` , :ref:`api-cmd-expireafter` , :ref:`api-cmd-hardpurge`.


.. _caching-policy-ttl:

Default TTL
---------------------

The TTL is set according to the response of the origin server.
Until the TTL is expired, saved content will be serviced.
When the TTL expires, send a check request for modified content to the origin server( **If-Modified-Since** or **If-None-Match** ).
The TTL will be extended if the origin server replies **304 Not Modified** to the request. ::

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
    
Except ``Ratio`` (0~100), all units are in seconds.

-  ``<Res2xx> (default: 1800 seconds, Ratio: 20, Max=86400)``
   If the origin server returns "200 OK", set the TTL.
   When saving content, expiration period is set to ``<Res2xx>`` seconds. 
   (After the TTL expires)If the origin server replies that the content has not been changed(304 Not Modified), the TTL can be extended according to the ``Ratio`` value(0~100).
   The TTL can be extended up to ``Max`` seconds.

-  ``<NoCache> (default: 5 seconds, Ratio: 0, Max=5, MaxAge=0)``
   This function works just like ``<Res2xx>``, but is only used when the origin server replies with "no-cache". ::

      cache-control: no-cache or private or must-revalidate
    
   If ``MaxAge`` is greater than 0, max-age can be applied.
    
   .. figure:: img/nocache_maxage.png
      :align: center

      Files are cached to the client for Max-Age seconds.

-  ``<Res3xx> (default: 300 seconds)``
   Set the TTL when the origin server replies with "3xx".
   This function is frequently used for redirect.
   
-  ``<Res4xx> (default: 30 seconds)``
   Set the TTL when the origin server replies with "4xx".
   In most cases replies are **404 Not Found**.
   
-  ``<Res5xx> (default: 30 seconds)``
   Set the TTL when the origin server replies with "5xx".
   This case usually comes with an internal error of the origin server.
   
-  ``<ConnectTimeout> (default: 3 seconds)``
   When the origin server is unable to be reached, set the TTL.
   If content is already saved, prolong the TTL for ``<ConnectTimeout>`` seconds.
   If content is not saved, keep the error status for ``<ConnectTimeout>`` seconds.
   This remedy is to reduce burden on the origin server that might be in a failed status rather than serving failed content.
   
-  ``<ReceiveTimeout> (default: 3 seconds)``
   Set the TTL when the connection is successful, but data acquisition is failing.
   This is similar to ``<ConnectTimeout>`` in its intention.

-  ``<OriginBusy> (default: 3 seconds)``
   If :ref:`origin-busysessioncount` condition is satisfied, prolong the TTL of expired contents by configured amount of time without requesting to the origin server.
   In this way, the origin server can avoid workload.
   
.. note::

   If 0 is set for the TTL content will expire as soon as it is serviced.
   A bypass is recommended if the origin server needs to reply to all requests.
   

.. _caching-policy-customttl:
   
Custom TTL
---------------------

This section explains how to set separate TTLs for each URL.
Specific TTLs can be set for matching content of specified URLs or patterned URLs.
The configuration is saved at /svc/{virtual host name}/ttl.txt. ::

    # /svc/www.example.com/ttl.txt
    # An identifier is comman(,), and the unit of time is second.
    
    *.jsp, 10
    /,5
    /index.html, 5
    /script/*.js, 300
    /image/ad.jpg, 1800
    

Even if you add *.html in order to set separate TTLs for all pages(html, php, jsp, etc.), the TTL will not be set for the first page(/). 
The HTTP protocol cannot identify which page(i.e. index.php, default.jsp, etc.) is set as the first page by the origin server.
Therefore, in order to set the separate TTL for each page, you should add "/".

    
TTL Priority
---------------------

This section explains how to decide which TTL configuration is applied first. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTL Priority="cc_nocache, custom, cc_maxage, rescode">
        ... (skipped) ...
    </TTL>    
    
The priority can be configured in the ``Priority (default: cc_nocache, custom, cc_maxage, rescode)`` item of the ``<TTL>`` tag.

- ``cc_nocache`` When the origin server replies "Cache-Control: no-cache"
- ``custom`` `caching-policy-customttl`
- ``cc_maxage`` When the origin server specifies maxage in Cache-Control
- ``rescode`` The default TTL related to the response code of the origin server


Abnormal TTL Extension
---------------------

If the origin server intermittently produces an error response, it might be better if it just totally fails.
For example, the server may lose connection with the storage that saves content, or it might decide regular service is unavailable.
The server will reply with a 4xx response(usually **404 Not Found**) for the former case, and it will reply with a 5xx response(usually **500 Internal Error**) for the latter case.

On the other hand, if related content is already saved in the storage,
it would be more efficient to extend the TTL to keep the service running rather than relying on responses from the origin server. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTLExtensionBy4xx>OFF</TTLExtensionBy4xx>
    <TTLExtensionBy5xx>ON</TTLExtensionBy5xx>

-  ``<TTLExtensionBy4xx>``

   -  ``OFF (default)`` Updates content with a 4xx reply.
   
   -  ``ON`` Reacts as if it received **304 Not Modified** as a reply.
   
You should also be aware of the intended 4xx response.
       
-  ``<TTLExtensionBy5xx>``

   -  ``ON (default)`` Reacts as if it received **304 Not Modified** as a reply.
   
   -  ``OFF`` Updates content with a 5xx reply.

A normal server does not return 5xx as a reply, as it is used for reducing the load on the origin server by invalidating contents from a temporary server error.
    

.. _caching-policy-renew:

Renewal Policy
====================================

TTL expired content is serviced after modifications are checked at the origin server.

   .. figure:: img/perf_refreshexpired.jpg
      :align: center
      
      Reply after modification check

1. The TTL is valid. 
   Reply immediately.

#. The TTL is expired and requests a modification check(If-Modified-Since) from the origin server. 
   Does not respond to the client until hearing from the server.
   
#. If the origin server replies, extend the TTL or swap content. 
   Respond to the client since the origin server confirmed.
   
#. The content that has been checked for modification will be serviced immediately until the next TTL expiration period.

Services that emphasize transfer speed rather than reactivity, such as high quality videos or games, do not care about this service method.
Bulk data will not care even if the origin server takes 10 seconds to respond to the modification check request because the entire transfer time is much longer.
Therefore, the reactivity of the origin server is not as important as other services.
Rather this content needs modification checks because it is not accessed frequently.

Online shopping malls, on the other hand, are different. 
The loading speeds of webpages are more important than anything else. 
The webpage has to be fully loaded within a few seconds. 
In this case, reactivity is more important than transfer speed.

A huge delay could occur when a client opens a web page at the moment the TTL expires and requests a modification check. 
Considering the fact that most shopping malls service millions of content simultaneously, it would be better to check modifications all the time.
The worst cases are when the origin server fails or the network is disconnected.

What you want is to stably transfer cached content regardless of any origin server error or delay.

   .. figure:: img/perf_refreshexpired2.jpg
      :align: center
      
      No fear of errors!
      
These different requirements leads to the development of a background content update. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <RefreshExpired>ON</RefreshExpired>    

-  ``<RefreshExpired>``

   -  ``ON (default)`` Replies after a modification check
   
   -  ``OFF`` Replies without the response of a modification check
      When the new content download is completed, the new content swaps with the old content.

``OFF`` This option is generally used because content is not frequently modified.

   .. figure:: img/perf_refreshexpired5.jpg
      :align: center
      
      If the content is not sensitive to modifications, it will not wait for a response from the origin server.

In the above figure, the content update of the origin server is running in the background, so cached content can be serviced to clients immediately.
If the origin server replies **304 Not Modified** then the TTL is simply extended.
If the origin server replies 200 OK due to a file update, the file is seamlessly replaced after the download is completed.
Clients who were downloading previous content (green bar) will not have any problems even if the content has been updated. 
Clients who access  the updated content (yellow bar) will be served with new content. 
Because content updates are always running in the background, the service is not affected by any hiccups such as contents updates, network failures and origin server errors. 


TTL Expiration When No-cache is Requested by Clients
---------------------

If there is at least one no-cache setting in the client HTTP request, related content can be expired right away. ::

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

   -  ``OFF (default)`` Ignores
   
   -  ``ON`` Expires the TTL right away
   
Expired contents abide by the `Expiration Policy`_.


Accept-Encoding Header
====================================

Even if there is an HTTP request for identical URLs, the existence of an Accept-Ending header affects which content will be cached.
When STON sends a request to the origin server, STON does not have information about whether the file is compressed or not.
Even if STON received compressed information, the information cannot be compared for every request.

   .. figure:: img/acceptencoding.png
      :align: center

      It is hard to expect a reply from the origin server.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <AcceptEncoding>ON</AcceptEncoding>    

-  ``<AcceptEncoding>``

   -  ``ON (default)`` Recognizes the Accept-Ending header from the HTTP client
   
   -  ``OFF`` Ignores the Accept-Ending header from the HTTP client
    
If the origin server does not support compression or a bulk file that does not require compression, it is recommended to set ``<AcceptEnding>`` to ``OFF``.


.. _caching-policy-casesensitive:

Identifying Upper / Lower Case Letters
====================================

STON cannot identify whether the origin server recognizes upper or lower case letters.

   .. figure:: img/casesensitive.png
      :align: center

      It might be the same content or else a 404 Error will occur.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <CaseSensitive>ON</CaseSensitive>    

-  ``<CaseSensitive>``

   -  ``ON (default)`` Distinguishes URL upper/lower case letters. 
   
   -  ``OFF`` Processes all URL letters to lower case letters.

    
.. _caching-policy-applyquerystring:
    
Identifying QueryString
====================================

Identifying the query string is not necessary unless the content is dynamically created by the query string.
If a URL contains a meaningless random value or a constantly changing time value, it will cause a huge load on the origin server.

   .. figure:: img/querystring.png
      :align: center

      Most likely it is identical content unless it is not dynamic content.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <ApplyQueryString>ON</ApplyQueryString>    

-  ``<ApplyQueryString>``

   -  ``ON (default)`` Identifies query strings. If the exception condition is met, the query string will be ignored.
   
   -  ``OFF`` Ignores query strings. If the exception condition is met, query strings will be identified.
    
Query string exceptions are saved at /svc/{virtual host name}/querystring.txt. ::

    # ./svc/www.example.com/querystring.txt
    
    /private/personal.jsp?login=ok*
    /image/ad.jpg

Please be aware that whether the exception is ignored or accepted depends on the ``<ApplyQueryString>`` setting. 
Distinctive or patterned URLs (* is only allowed) can be used in the configuration.


Vary Header (재검토 대상)
====================================

Content is identified by the Vary header. (재번역 요망: 의미가 다릅니다. 같은 컨텐츠라도 다른 vary 헤더가 붙어서 오면 다른 컨텐츠로 인식될 수 있습니다. 자세히 풀어서 말하면 이렇습니다. STON이 컨텐츠를 원본에서 받아 사용자에게 전달하죠? 원본에서 컨텐츠를 받을 때 같이 오는 Vary 헤더들 중에 사용자에게 전달할지 씹을지를 구분한다는 말입니다. 같은 컨텐츠라도 vary 헤더에 따라 다른 컨텐츠로 받아 들여질 수 있습니다 )
In most cases the Vary header causes a huge performance drop of the cache server. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader />    
    
-  ``<VaryHeader>``

   Configures the supported header list from Vary headers that the origin server responded.
   Comma(,) is used as an identifier.

For example, even if the origin server sends the following Vary header, it will be ignored if ``<VaryHeader>`` is not configured. ::

    Vary: Accept-Encoding, Accept, User-Agent

In order to identify Accept-Encoding and Accept headers (except the User-Agent header, set as shown below. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>Accept-Encoding, Accept</VaryHeader>    
    
In order to identify all Vary headers from the origin server, set as shown below. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>*</VaryHeader>    


POST Request
====================================

The following explains the configuration to cache a POST request. 
A POST request has an identical URL with different Body data. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <PostRequest MaxContentLength="102400" BodySensitive="ON">OFF</PostRequest>    

-  ``<PostRequest>``

   -  ``OFF (default)`` Terminates the session when a POST request is received
   
   -  ``ON`` Caches POST requests.
   
Most POST request processing cases use Body data as a Caching-Key.
Detailed configuration is available with ``BodySensitive`` property and exceptions.

-  ``BodySensitive``

    -  ``ON (default)`` Identifies Body data as a Caching-Key
       Maximum length will be limited by the ``MaxContentLength (default: 102400 Bytes)`` property.
       If the exception is met, Body data will be ignored.
    
    -  ``OFF`` Ignores Body data 
       If the exception is met, identifies Body data.
   
A POST request exception is saved at /svc/{virtual host name}/postbody.txt. ::
    
    # /svc/www.example.com/postbody.txt
    
    /bigsale/*.php?nocache=*
    /goods/search.php
    
Please be aware that whether the exception is ignored or accepted depends on the ``<BodySensitive>`` setting.
Distinctive or patterned URLs (* is only allowed) can be used in the configuration. 
  
.. note::

    If the ``MaxContentLength`` value is set too high, more memory will be required for managing the Caching-Key.
    It is recommended to set it as small as possible.

