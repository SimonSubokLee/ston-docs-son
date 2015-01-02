.. _caching-policy:

Chapter 4. Caching Policy
******************

This chapter will explain TTL(Time To Live), Caching-Key and expiration policy that are fundamental to the service.
Stored contents are only available while TTL is valid.
Standard HTTP protocol specifies Cache-Control for setting the TTL.
하지만 이는 절대적인 것은 아니다(무엇이 어떻게 절대적인 것이 아닌지???). 
Various TTL policies and :ref:`caching-purge` will improve service quality.

HTTP has various standards that classify contents. 
Also, there could be various Caching-Key as well. 
Less frequently updating contents not only help reducing load on the origin server but also easy to scale up.
콘텐츠 변경이 없을수록 원본부하를 줄일 수 있을뿐만 아니라 (무엇을 쉽게 확장할 수 있는지??) 쉽게 확장할 수 있다.
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

고화질 동영상이나 게임처럼 상대적으로 반응성보다 전송속도가 중요한 서비스에서는 이런 방식이 큰 문제가 되지 않는다. 
대용량 데이터의 경우 원본서버가 10초만에 갱신에 대한 응답을 한다고 하더라도 전송에 몇 분씩 소요되기 때문에 상대적으로 원본의 반응성이 크게 중요하지 않다. 
오히려 접근 빈도가 높지 않은 콘텐츠이기 때문에 반드시 갱신확인이 이루어져야 한다.

하지만 쇼핑몰과 같은 경우 상황은 다르다. 
웹 페이지는 빠르게 로딩되는 것이 무엇보다 중요하다. 
1~2초 안에 클라이언트 화면구성이 모두 끝나야 한다.
전송속도보다 반응속도가 더 중요하다는 말이다. 

이때 TTL이 만료되어 원본서버에게 갱신확인을 해야 한다면 매우 큰 지연이 발생할 수 있다. 
보통의 쇼핑몰이 수백만개의 콘텐츠를 동시에 서비스 하는 것을 감안할 때 항상 원본서버로부터 갱신확인 작업이 발생하고 있다고 생각해야 한다. 
자칫 원본서버 장애가 발생하거나 네트워크 장애가 발생한다면 최악이다.

우리가 원하는 것은 어떠한 원본서버의 장애나 지연으로부터 캐싱된 콘텐츠가 안전하게 전송되는 것이다.

   .. figure:: img/perf_refreshexpired2.jpg
      :align: center
      
      장애가 두렵지 않다!
      
이런 차이점 때문에 백그라운드 콘텐츠 갱신기능이 개발되었다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <RefreshExpired>ON</RefreshExpired>    

-  ``<RefreshExpired>``

   -  ``ON (기본)`` 변경확인 후 응답한다.
   
   -  ``OFF`` 변경확인 응답을 기다리지 않고 응답한다.
      새로운 콘텐츠의 다운로드가 완료되면 그때 변경(Swap)한다.

``OFF`` 설정의 더 큰 이유는 콘텐츠가 대부분 자주 바뀌지 않기 때문이다.

   .. figure:: img/perf_refreshexpired5.jpg
      :align: center
      
      변경에 민감하지 않다면 기다리지 않는다.

위 그림에서 원본서버와의 갱신작업이 모두 백그라운드로 이루어지기 때문에 캐싱된 콘텐츠는 기다림없이 즉시 클라이언트에게 서비스된다. 
원본서버가 **304 Not Modified** 로 응답한다면 TTL만 연장된다. 
파일이 갱신되어 원본서버에서 200 OK를 응답한 경우 해당 파일이 완전히 다운로드된 후에 파일이 부드럽게 교체된다. 
콘텐츠이 바뀌어도 이전 콘텐츠(초록색)를 다운로드 받던 사용자들은 정상적으로 다운로드가 진행된다. 
파일 교체 이후 접근된 사용자들(겨자색)은 바뀐 파일로 서비스 된다.
콘텐츠 갱신, 네트워크 장애, 원본서버 장애 등 어떠한 변수에도 콘텐츠 갱신은 백그라운드로 진행되기 때문에 실제 서비스에는 전혀 지연이 없다.


클라이언트 no-cache 요청시 TTL만료
---------------------

클라이언트 HTTP요청에 no-cache 설정이 하나 이상 명시된 경우 해당 콘텐츠를 즉시 만료시킬 수 있다. ::

    GET /logo.jpg HTTP/1.1
    ...
    cache-control: no-cache 또는 cache-control:max-age=0
    pragma: no-cache
    ...
    
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <NoCacheRequestExpire>OFF</NoCacheRequestExpire>    

-  ``<NoCacheRequestExpire>``

   -  ``OFF (기본)`` 무시한다.
   
   -  ``ON`` TTL을 즉시 만료한다.
   
만료된 콘텐츠는 `갱신정책`_ 에 따른다.


Accept-Encoding 헤더
====================================

같은 URL에 대한 HTTP요청이라도 Accept-Encoding헤더의 존재 유무에 따라 다른 콘텐츠가 캐싱될 수 있다. 
원본서버에 요청을 보내는 시점에 압축여부를 알 수 없다.
응답을 받았다고해도 압축여부를 매번 비교할 수도 없다.

   .. figure:: img/acceptencoding.png
      :align: center

      원본서버가 어떤 응답을 줄지 알 수 없다.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <AcceptEncoding>ON</AcceptEncoding>    

-  ``<AcceptEncoding>``

   -  ``ON (기본)`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 인식한다.
   
   -  ``OFF`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 무시한다.
    
원본서버에서 압축을 지원하지 않거나, 압축이 필요없는 대용량 파일의 경우 ``OFF`` 로 설정하는 것이 바람직하다.


.. _caching-policy-casesensitive:

대소문자 구분
====================================

원본서버의 대소문자 구분여부를 능동적으로 알 수 없다.

   .. figure:: img/casesensitive.png
      :align: center

      아마도 같은 콘텐츠이거나 404가 발생한다.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <CaseSensitive>ON</CaseSensitive>    

-  ``<CaseSensitive>``

   -  ``ON (기본)`` URL 대소문자를 구문한다. 
   
   -  ``OFF`` URL 대소문자를 구분하지 않는다. 모두 소문자로 처리된다.

    
.. _caching-policy-applyquerystring:
    
QueryString 구분
====================================

QueryString에 의하여 동적으로 생성되는 콘텐츠가 아니라면 QueryString을 인식하는 것은 불필요하다. 
아무 의미없는 Random값이나 항상 변하는 시간 값이 매번 붙는다면 원본에 엄청난 부하가 발생할 수 있다.

   .. figure:: img/querystring.png
      :align: center

      동적 콘텐츠가 아니라면 같은 콘텐츠일 가능성이 높다.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <ApplyQueryString>ON</ApplyQueryString>    

-  ``<ApplyQueryString>``

   -  ``ON (기본)`` QueryString을 인식한다. 예외조건에 만족하면 QueryString이 무시된다.
   
   -  ``OFF`` QueryString을 무시한다. 예외조건에 만족하면 QueryString을 인식한다.
    
QueryString-예외조건은 /svc/{가상호스트 이름}/querystring.txt에 설정한다. ::

    # ./svc/www.example.com/querystring.txt
    
    /private/personal.jsp?login=ok*
    /image/ad.jpg

예외조건이 ``<ApplyQueryString>`` 설정에 따라 의미가 달라짐에 주의한다. 
명확한 URL또는 패턴(*만 허용한다)으로 설정이 가능하다.


Vary 헤더
====================================

Vary헤더를 인식하여 콘텐츠를 구분한다. 
일반적으로 Vary헤더는 Cache서버의 성능을 급격히 떨어트리는 원흉이다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader />    
    
-  ``<VaryHeader>``

   원본서버가 응답한 Vary헤더 중 지원할 헤더목록을 설정한다.
   구분자는 콤마(,)를 사용한다.

예를 들어 원본서버가 다음과 같이 Vary헤더를 보냈다고 하더라도 ``<VaryHeader>`` 가 설정되어 있지 않다면 무시한다. ::

    Vary: Accept-Encoding, Accept, User-Agent

User-Agent를 제외한 Accept-Encoding과 Accept헤더만을 인식하도록 하려면 다음과 같이 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>Accept-Encoding, Accept</VaryHeader>    
    
원본서버가 보낸 모든 Vary헤더를 인식하게 하려면 다음과 같이 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>*</VaryHeader>    


POST 요청
====================================

POST 요청을 Caching하도록 설정한다. 
POST 요청의 특성상 URL은 같지만 Body데이터가 다를 수 있다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <PostRequest MaxContentLength="102400" BodySensitive="ON">OFF</PostRequest>    

-  ``<PostRequest>``

   -  ``OFF (기본)`` POST요청이 오면 세션을 종료한다.
   
   -  ``ON`` POST요청을 Caching한다.
   
실제로 POST요청을 처리하는 대부분의 경우는 Body데이터를 Caching-Key로 사용한다.
``BodySensitive`` 속성과 예외조건을 통해 정교한 설정이 가능하다.

-  ``BodySensitive``

    -  ``ON (기본)`` Body데이터까지 Caching-Key로 인식한다.
       최대 길이는 ``MaxContentLength (기본: 102400 Bytes)`` 속성으로 제한한다.
       예외조건에 만족하면 Body데이터를 무시한다.
    
    -  ``OFF`` Body데이터는 무시한다. 
       예외조건에 만족하면 Body데이터를 인식한다.
   
POST요청 예외조건은 /svc/{가상호스트 이름}/postbody.txt에 설정한다. ::
    
    # /svc/www.example.com/postbody.txt
    
    /bigsale/*.php?nocache=*
    /goods/search.php
    
예외조건이 ``BodySensitive`` 설정에 따라 의미가 달라짐에 주의한다. 
명확한 URL 또는 패턴(*만 허용한다.)으로 설정이 가능하다.
  
.. note::

    ``MaxContentLength`` 속성을 너무 크게 설정할 경우 Caching-Key 관리에 많은 메모리가 필요하다.
    가능한 작게 설정하는 것이 좋다.
    
