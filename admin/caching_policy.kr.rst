.. _caching-policy:

Chapter 4. Caching Policy
******************

This chapter will explain TTL(Time To Live), Caching-Key and expiration policy that are fundamental to the service.
Stored contents are only available while TTL is valid.
Standard HTTP protocol specifies Cache-Control for setting the TTL.
하지만 이는 절대적인 것은 아니다(무엇이 어떻게 절대적인 것이 아닌지???: 규격은 있지만 꼭 해야 하는것도 아니고 그렇게 되지도 않습니다). 
Various TTL policies and :ref:`caching-purge` will improve service quality.

HTTP has various standards that classify contents. 
Also, there could be various Caching-Key as well. 
Less frequently updating contents not only help reducing load on the origin server but also easy to scale up.
콘텐츠 변경이 없을수록 원본부하를 줄일 수 있을뿐만 아니라 (무엇을 쉽게 확장할 수 있는지??: 서비스를 쉽게 확장할수 있습니다) 쉽게 확장할 수 있다.
In this chapter, several methods to establish optimized expiration policy for a service will be discussed.

In order to use upcoming configuration as a default setting for all virtual hosts, place it under the ``<VHostDefault>``.
On the contrary, to use the configuration for the specific virtual host, place it under the <Vhost> tag.

**Caching-Key** is a unique value that distinguishes contents. 
In a similar way, file system has a unique path for a file(eg. /usr/conf.txt).
Occasionally people confuses Caching-Key with URL.
However, depends on various functions in HTTP, identical URL could return different contents.


.. toctree::
   :maxdepth: 2



TTL (Time To Live)
====================================

TTL stands for the expiration time of saved contents.
Longer TTL setting reduces burden of the origin server, but modifications will be applied after TTL is expired.
On the contrary, shorter TTL setting will increase load of the origin server because of frequent requests for modification check.
The beauty of TTL management is to find an appropriate setting for reducing origin server load.
Once TTL is set, it will not change until TTL is expired.
The new TTL will be applied when the old TTL for the file is expired.
Administrator can change TTL setting by using these APIs, :ref:`api-cmd-purge` , :ref:`api-cmd-expire` , :ref:`api-cmd-expireafter` , :ref:`api-cmd-hardpurge`.


.. _caching-policy-ttl:

Default TTL
---------------------

TTL is set according to the response of the origin server.
Until TTL is expired, saved contents will be serviced.
When TTL expires, send a check request for modified contents to the origin server( **If-Modified-Since** or **If-None-Match** ).
TTL will be extended if the origin server replies **304 Not Modified** for the request. ::

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
    
Except ``Ratio`` (0~100), all units are in second.

-  ``<Res2xx> (default: 1800 seconds, Ratio: 20, Max=86400)``
   If the origin server returns "200 OK", set the TTL.
   When saving contents, expiration period is set to ``<Res2xx>`` seconds. 
   (After TTL expired)If the origin server replies that the contents have not been changed(304 NOt Modified), TTL is extended according to the ``Ratio`` value(0~100).
   TTL can be extended up to ``Max`` seconds.

-  ``<NoCache> (default: 5 seconds, Ratio: 0, Max=5, MaxAge=0)``
   This function works just like ``<Res2xx>``, but only used when the origin server replies with "no-cache". ::

      cache-control: no-cache or private or must-revalidate
    
   If ``MaxAge`` is greater than 0, max-age can be applied.
    
   .. figure:: img/nocache_maxage.png
      :align: center

      Files are cached to the client for Max-Age seconds.

-  ``<Res3xx> (default: 300 seconds)``
   Set TTL when the origin server replies with "3xx".
   This function is frequently used for redirect.
   
-  ``<Res4xx> (default: 30 seconds)``
   Set TTL when the origin server replies with "4xx".
   In most cases replies are **404 Not Found**.
   
-  ``<Res5xx> (default: 30 seconds)``
   Set TTL when the origin server replies with "5xx".
   This case usually comes with an internal error of the origin server.
   
-  ``<ConnectTimeout> (default: 3 seconds)``
   When the origin server is unable to reach, set TTL.
   If contents are already saved, prolong TTL for ``<ConnectTimeout>`` seconds.
   If contents are not saved, keep the error status for ``<ConnectTimeout>`` seconds.
   This remedy is to reduce burdens of the origin server that might be in a failed status rather than serving failed contents.
   
-  ``<ReceiveTimeout> (default: 3 seconds)``
   Set TTL when the connection is successful, but data acquisition is failing.
   This is similar to ``<ConnectTimeout>`` in its intention.

-  ``<OriginBusy> (default: 3 seconds)``
   If :ref:`origin-busysessioncount` condition is satisfied, prolong TTL of expired contents by configured amount of time without requesting to the origin server.
   In this way, the origin server can avoid workload.
   
.. note::

   If 0 is set for TTL, contents will be expired as soon as they are serviced.
   Bypass is recommended if the origin server needs to reply all requests.
   

.. _caching-policy-customttl:
   
Custom TTL
---------------------

This section explains how to set separate TTLs for each URL.
Specific TTL can be set for matching contents of specified URL or patterned URL.
Configuration is saved at /svc/{virtual host name}/ttl.txt. ::

    # /svc/www.example.com/ttl.txt
    # An identifier is comman(,), and the unit of time is second.
    
    *.jsp, 10
    /,5
    /index.html, 5
    /script/*.js, 300
    /image/ad.jpg, 1800
    

Even if you added *.html in order to set separate TTLs for all pages(html, php, jsp etc), TTL will not be set for the first page(/). 
HTTP protocol cannot identify which page(eg. index.php, default.jsp etc) is set as the first page by the origin server.
Therefore, in order to set the separate TTL for each page, you should add "/".

    
TTL Priority
---------------------

This section explains how to decide which TTL configuration is applied first. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTL Priority="cc_nocache, custom, cc_maxage, rescode">
        ... (skipped) ...
    </TTL>    
    
The priority can be configured in the ``Priority (default: cc_nocache, custom, cc_maxage, rescode)`` item of ``<TTL>`` tag.

- ``cc_nocache`` When the origin server replies "Cache-Control: no-cache"
- ``custom`` `caching-policy-customttl`
- ``cc_maxage`` When the origin server specifies maxage in Cache-Control
- ``rescode`` Default TTL related to the response code of the origin server


Abnormal TTL Extension
---------------------

When the origin server intermittently responses under the error situation, it could be much better if it just totally fails.
For example, the server loses connection with the storage that saves contents or it might decide regular service is unavailable.
The server will reply 4xx response(usually **404 Not Found**) for the former case, and it will reply 5xx response(usually **500 Internal Error**) for the latter case.

On the other hand, if related contents are already saved in the storage,
it would be more efficient to extend TTL to keep the service running rather than relying on responses from the origin server. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTLExtensionBy4xx>OFF</TTLExtensionBy4xx>
    <TTLExtensionBy5xx>ON</TTLExtensionBy5xx>

-  ``<TTLExtensionBy4xx>``

   -  ``OFF (default)`` Update contents with 4xx reply.
   
   -  ``ON`` React as if it received 304 not modified as a reply.
   
You should be aware of intended 4xx reponse.
       
-  ``<TTLExtensionBy5xx>``

   -  ``ON (default)`` React as if it received **304 Not Modified** as a reply.
   
   -  ``OFF`` Update contents with 5xx reply.

A normal server does not return 5xx as a reply. 
It is used for reducing the load of origin server by invalidating contents from temperary error of the server.
    

.. _caching-policy-renew:

Renewal Policy
====================================

TTL expired contents are serviced after checking modifications at the origin server.

   .. figure:: img/perf_refreshexpired.jpg
      :align: center
      
      Reply after modification check

1. TTL is valid. 
   Reply immediately.

#. TTL is expired and requests modification check(If-Modified-Since) to the origin server. 
   Do not respond to the client until hear from the server.
   
#. If the origin server replies, extend TTL or swap contents. 
   Respond to the client since the origin server confirmed.
   
#. The contents that have been checked modification will be serviced immediately until the next TTL expiration period.

Services that emphasize transfer speed rather than reactivity such as high quality videos or games do not care about this service method.
Bulk data will not care even if the origin server takes 10 seconds to respond for the modification check request because the entire transfer time is much longer.
Therefore, the reactivity of origin server is not as important as other services.
These contents rather need modification check because they are not accessed frequently.

On the other hand, online shopping malls are different. 
The loading speeds of webpages are important more than anything else. 
The webpage has to be fully loaded within few seconds. 
In this case, reactivity is more important than trasnfer speed.

Huge delay could occur when a client opens a web page at the moment TTL is expired and requesting modification check. 
Considering the fact that usual shopping malls service millions of contents simultaneously, it would be better to check modifications all the time.
The worst cases are when the origin server fails or network is diconnected.

What you want is to stably transfer cached contents regardless of any origin server error or delay.

   .. figure:: img/perf_refreshexpired2.jpg
      :align: center
      
      No fear of errors!
      
These different requirments led to the development of background contents update. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <RefreshExpired>ON</RefreshExpired>    

-  ``<RefreshExpired>``

   -  ``ON (default)`` Reply after modification check.
   
   -  ``OFF`` Reply without the response of modification check.
      When the new content download is completed, swap with the old content.

``OFF`` 설정의 더 큰 이유(OFF 설정을 기본인 ON보다 더 많이 쓴다는 뜻인가요??: 네 OFF 설정이 일반적입니다)는 콘텐츠가 대부분 자주 바뀌지 않기 때문이다.

   .. figure:: img/perf_refreshexpired5.jpg
      :align: center
      
      변경에 민감하지 않다면 (서버로부터 변경 확인을??: 원본서버로부터)기다리지 않는다.

In the above figure, contents update of the origin server is running on the background, so cached contents can be serviced to clients immediately.
If the origin server replies **304 Not Modified** then simply TTL is extended.
If the origin server replies 200 OK due to file update, the file is seamlessly replaced after the download is completed.
Clients who were downloading previous contents(green bar) will not have any problems even if the contents have been updated. 
Clients who access to the updated contents(yellow bar) will be serviced with new contents. 
Contents updates are always running on the background, the service does not affected by any hiccups such as contents update, network failure and origin server error. 


TTL Expiration when no-cache Requested by Clients
---------------------

If there is at least one no-cache setting in the client HTTP request, related contents can be expired right away. ::

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

   -  ``OFF (default)`` Ignores.
   
   -  ``ON`` Expires TTL right away.
   
Expired contents abide by the `Expiration Policy`_.


Accept-Encoding Header
====================================

Even if there is a HTTP request for identical URL, the existence of Accept-Ending header affects which contents to be cached.
At the moment sending a request to the origin server, 
같은 URL에 대한 HTTP요청이라도 Accept-Encoding헤더의 존재 유무에 따라 다른 콘텐츠가 캐싱될 수 있다. 
원본서버에 요청을 보내는 시점에 (누가 무엇에 대한 압축여부를 확인하는지??: STON이 원본서버에 요청하는 시점을 말합니다)압축여부를 알 수 없다.
응답을 받았다고해도 압축여부를 매번 비교할 수도 없다.

   .. figure:: img/acceptencoding.png
      :align: center

      It is hard to expect the reply from the origin server.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <AcceptEncoding>ON</AcceptEncoding>    

-  ``<AcceptEncoding>``

   -  ``ON (default)`` Recognize Accept-Ending header from the HTTP client.
   
   -  ``OFF`` Ignore Accept-Ending header from the HTTP client.
    
If the origin server does not support compression or bulk file that does not require compression, it is recommended to set ``<AcceptEnding>`` to ``OFF``.


.. _caching-policy-casesensitive:

Identifying Upper / Lower Case Letters
====================================

(누가?? STON이??: 네) 원본서버의 대소문자 구분여부를 능동적으로 알 수 없다.

   .. figure:: img/casesensitive.png
      :align: center

      They might be the same contents or else 404 error will occur.
   
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

Identifying the queryString is not necessary unless the contents is dynamically created by the querystring.
If a URL contains a meaningless random value or a constantly changing time value, it will cause huge load on the origin server.

   .. figure:: img/querystring.png
      :align: center

      Most likely they are identical contents unless it is not a dynamic contents.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <ApplyQueryString>ON</ApplyQueryString>    

-  ``<ApplyQueryString>``

   -  ``ON (default)`` Identifies queryString. If exception condition is met, querystring will be ignored.
   
   -  ``OFF`` Ignores queryString. If exception condition is met, querystring will be identified.
    
QueryString exceptions are saved at /svc/{virtual host name}/querystring.txt. ::

    # ./svc/www.example.com/querystring.txt
    
    /private/personal.jsp?login=ok*
    /image/ad.jpg

Please be aware that the exception can work in a different way depends on the ``<ApplyQueryString>`` setting.
예외조건이 ``<ApplyQueryString>`` 설정에 따라 (예외조건의 의미?? 예외조건의 조건??: querystring.txt에 설정하는 조건은 <ApplyQueryString> 설정에 따라 무시(예외)될수도 인식(포함)될수도 있습니다)의미가 달라짐에 주의한다. 
Distinctive or patterned URL(* is only allowed) can be used in the configuration.


Vary Header
====================================

Contents are identified by the Vary header. 
In most cases Vary header causes huge performance drop of cache server. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader />    
    
-  ``<VaryHeader>``

   Configure the header list to support among Vary headers that the origin server responded.
   Comma(,) is used as an identifier.

For example, even if the origin server sent the following Vary header, it will be ignored if ``<VaryHeader>`` is not configured. ::

    Vary: Accept-Encoding, Accept, User-Agent

In order to identify Accept-Encoding and Accept header except User-Agent, set as belows. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>Accept-Encoding, Accept</VaryHeader>    
    
In order to identify all Vary header from the origin server, set as belows. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>*</VaryHeader>    


POST Request
====================================

Configure to cache POST request. 
POST request has an identical URL with different Body data. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <PostRequest MaxContentLength="102400" BodySensitive="ON">OFF</PostRequest>    

-  ``<PostRequest>``

   -  ``OFF (default)`` Terminate the session when POST request is received.
   
   -  ``ON`` Cache POST request.
   
Most POST request processing cases use Body data as a Caching-Key.
Detailed configuration is available with ``BodySensitive`` property and exceptions.

-  ``BodySensitive``

    -  ``ON (default)`` Identifies Body data as a Caching-Key.
       Maximum length will be limited by ``MaxContentLength (default: 102400 Bytes)`` property.
       If exception is met, Body data will be ignored.
    
    -  ``OFF`` Ignore Body data. 
       If exception is met, identifies Body data.
   
POST request exception is saved at /svc/{virtual host name}/postbody.txt. ::
    
    # /svc/www.example.com/postbody.txt
    
    /bigsale/*.php?nocache=*
    /goods/search.php
    
Please be aware that the exception can work in a different way depends on the ``<BodySensitive>`` setting.
Distinctive or patterned URL(* is only allowed) can be used in the configuration.
예외조건이 ``BodySensitive`` 설정에 따라 (예외조건의 의미?? 예외조건의 조건??: postbody.txt에 설정하는 조건은 <BodySensitive> 설정에 따라 무시(예외)될수도 인식(포함)될수도 있습니다 )의미가 달라짐에 주의한다. 
  
.. note::

    If ``MaxContentLength`` value is set to high, more memory will be required for managing the Caching-Key.
    It is recommended to set it as small as possible.

