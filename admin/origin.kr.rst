.. _origin:

Chapter 7. Origin Server
******************

This chapter will explain the relationship between STON and the origin server.
The origin server generally stands for the web server that abides by HTTP standard.
Administrator should have thorough understanding about the contents in this chapter in order to protect the origin server.
After understanding this chapter, you can establish durable and flexible service that can resist from the origin server error.

The origin server has to be protected.
There are variety of plans for dealing with various errors.
Proper protection policy for the origin server will let you have relaxed server inspection.


.. toctree::
   :maxdepth: 2



.. _origin_exclusion_and_recovery:

Error Detection and Recovery
====================================

If failure occurs to the origin server during caching, the server will be automatically excluded.
When the server is recovered, it'll be utilized for the service. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
    
   <ConnectTimeout>3</ConnectTimeout>
   <ReceiveTimeout>10</ReceiveTimeout>
   <Exclusion>3</Exclusion>
   <Recovery Cycle="10" Uri="/" ResCode="0" Log="ON">5</Recovery>   

-  ``<ConnectTimeout> (default: 3 seconds)``
   
   If the origin server is not connected within the set amount of second, it is considered as a connection failure.

-  ``<ReceiveTimeout> (default: 10 seconds)``
   
   If the origin server does not reply HTTP response for the set amount of second for a normal HTTP request, it is considered as transaction failure.

-  ``<Exclusion> (default: 3 times)``
   
   If the set amount of consecutive failures( ``<ConnectTimeout>`` or ``<ReceiveTimeout>`` ) occur in the origin server, related server will be excluded from the available server list. 
   This value will be reset to 0 if a successful communication occurs before exclusion.

-  ``<Recovery> (default: 5 times)``
   
   Request with ``Uri`` in very ``Cycle``, and if the origin server replies ``ResCode`` for the set amount of consecutive times, then recover related server.
   Setting this value to 0 will not recover the server.   
   
   -  ``Cycle (default: 10 seconds)`` Try request every configured seconds.
   
   -  ``Uri (default: /)`` Uri to send request.
   
   -  ``ResCode (default: 0)`` Response code that will be identified as a normal response.
      Setting this value to 0 will regard any reply as a success.
      For example, setting this value to 200 will only process 200 response as a success.
      Multiple valid response codes can be configured by using comma(,).
      For example, 200, 206, 404 value will regard one of these responses as a success.

   -  ``Log (default: ON)`` Record HTTP Transaction that was used for recovery to :ref:`admin-log-origin`.
      
      

.. _origin-health-checker:

Health-Checker
====================================

`Error Detection and Recovery`_ responds to the failure during Caching process.
``<Recovery>`` terminates HTTP Transaction as soon as receiving a response code.
However, Health-Checker checks for a successful HTTP Transaction. ::

   # vhosts.xml - <Vhosts><Vhost>
   
   <Origin>
      <Address> ... </Address>
      <HealthChecker ResCode="0" Timeout="10" Cycle="10" 
                     Exclusion="3" Recovery="5" Log="ON">/</HealthChecker>
      <HealthChecker ResCode="200, 404" Timeout="3" Cycle="5" 
                     Exclusion="5" Recovery="20" Log="ON">/alive.html</HealthChecker>   
   </Origin>

-  ``<HealthChecker> (default: /)``

   Configures Health-Checker. Multiple configurations are allowed.
   Use Uri as a value, and CDATA is used for XML exception characters.
   
   -  ``ResCode (default: 0)`` Correct response codes (Multiple configurations are allowed with commas(,))
   
   -  ``Timeout (default: 10 seconds)`` A valid time from socket connection till complete HTTP Transaction.

   -  ``Cycle (default: 10 seconds)`` An execution period.
   
   -  ``Exclusion (default: 3 times)`` A number of times to fail before excluding related server.
   
   -  ``Recovery (default: 5 times)`` A number of consecutive successes before deploying the server.
   
   -  ``Log (default: ON)`` Records HTTP Transaction to :ref:`admin-log-origin` .

Health-Checker can have multiple configurations, and executed independently regardless of client requests.
It does not share information with the `Error Detection and Recovery`_ or other Health-Checkers to decide exclusion and deployment.


.. _origin-use-policy:

Origin Address 원본주소 사용정책
====================================

원본주소(IP)는 다음 요소들에 의해 어떻게 사용될지 결정된다.

-  :ref:`env-vhost-activeorigin` 주소 형식(IP 또는 Domain)과 보조주소
-  `장애감지와 복구`_
-  `Health-Checker`_

서비스를 운영하다보면 원본주소가 배제/복구되는 일은 빈번하다. 
STON은 IP테이블을 기반으로 원본주소를 사용하며 `origin-status`_ API를 통해 정보를 제공한다.

원본주소를 IP로 설정한 경우 매우 간단하다. 

-  설정변경 이외에 IP목록을 변화시키는 요인은 없다.
-  TTL에 의해 IP주소가 만료되지 않는다.
-  장애/복구 모두 설정(IP주소)에 기반하여 동작한다.

원본주소를 Domain으로 설정하면 Resolving해서 IP를 얻어야 한다.
( :ref:`admin-log-dns` 에 기록된다.)
IP 목록은 동적으로 변경될 수 있으며 모든 IP는 TTL(Time To Live)동안만 유효하다.

-  Domain은 주기적으로(1~10초) Resolving한다.
-  Resolving결과를 통해 사용할 IP테이블을 구성한다.
-  모든 IP는 TTL만큼만 유효하며 TTL이 만료되면 사용하지 않는다.
-  같은 IP가 다시 Resolving되면 TTL을 갱신한다.
-  IP테이블은 비어서는 안된다. (TTL이 만료되었더라도) 마지막 IP들은 삭제되지 않는다.

원본주소를 Domain으로 설정하여도 장애/복구는 IP기반으로 동작한다. 
여기서 미묘한 점이 있다.
DNS 클라이언트(=STON)는 Domain의 모든 IP 목록을 정확히 알 수 없다. 
하지만 사용할 수 없는 IP들만으로 Domain을 구성할 경우 장애상태가 지속될 수 없다.

Domain주소 장애/복구 정책은 다음과 같다.

-  (Domain에 대해) 알고 있는 모든 IP주소가 배제(Inactive)되면 해당 Domain주소가 배제된다.
-  신규 IP가 Resolving되더라도 Domain이 배제되어 있다면 IP주소는 처음부터 배제된다.
-  모든 IP가 TTL 만료되더라도 배제된 Domain상태는 풀리지 않는다.
-  배제된 Domain에 속한 IP주소가 하나라도 복구되어야 해당 Domain은 다시 활성화된다.

다소 복잡한 내용이므로 `origin-status`_ API를 통해 서비스 동작상태에 대해 이해도를 높이는 것이 좋다.



.. _origin-status:

원본상태 모니터링
====================================

API를 통해 가상호스트의 원본상태를 모니터링한다. ::

   http://127.0.0.1:10040/monitoring/origin       // 모든 가상호스트
   http://127.0.0.1:10040/monitoring/origin?vhost=www.example.com
   
결과는 JSON형식으로 제공된다. ::

   {
       "origin" : 
       [
           {
               "VirtualHost" : "example.com", 
               "Address" : 
               [ 
                   { "1.1.1.1" : "Active" },
                   { "1.1.1.2" : "Active" }
               ], 
               "Address2" : [  ], 
               "ActiveIP" : 
               [ 
                   { "1.1.1.1" : 0 },
                   { "1.1.1.2" : 0 }
               ] , 
               "InactiveIP" : [ ]
           },
           {
               "VirtualHost" : "foobar.com", 
               "Address" : 
               [
                   { "origin.foobar.com" : "Active" }
               ], 
               "Address2" : [  ], 
               "ActiveIP" : 
               [
                   { "5.5.5.5" : 21 },
                   { "5.5.5.6" : 60 },
                   { "5.5.5.7" : 37 }
               ], 
               "InactiveIP" :
               [
                   { "5.5.5.8" : 10 },
                   { "5.5.5.9" : 184 }
               ]  
           }
       ]
   }
    
-  ``VirtualHost`` 가상호스트 이름

-  ``Address`` :ref:`env-vhost-activeorigin` .
   설정주소가 사용중이라면 ``Active`` , (장애발생으로) 사용하고 있지 않다면 ``Inactive`` 로 표시된다.

-  ``Address2`` :ref:`env-vhost-standbyorigin` .
   설정주소를 사용중이라면 ``Active`` , 사용하고 있지 않다면 ``Inactive`` 로 표시된다.

-  ``ActiveIP`` 사용 중인 IP목록과 TTL. 
   원본서버를 IP로 설정하면 ``Address`` 와 동일한 IP에 TTL은 0으로 표시된다.
   Domain으로 설정하면 Resolving결과에 따른다.
   다양한 IP와 TTL을 사용한다.
   
-  ``InactiveIP`` 사용하지 않는 IP목록과 TTL.
   사용하지 않더라도 복구 중이거나 HealthChecker에 의해 관리될 수 있다.
   해당 주소는 TTL 동안 복구되지 않으면 삭제된다.
   

    
.. _origin-status-reset:

원본상태 초기화
====================================

API를 통해 가상호스트의 원본서버 배제/복구를 초기화한다. 
또한 현재 사용 중인 세션을 재사용하지 않고 새롭게 연결을 생성한다. ::

   http://127.0.0.1:10040/command/resetorigin       // 모든 가상호스트
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com   



.. _origin-busysessioncount:

과부하 판단
====================================

처음 요청되는 콘텐츠는 항상 원본서버에 요청해야 한다.
하지만 이미 Caching된 콘텐츠라면 좀 더 유연하게 대처할 수 있다.
원본서버가 과부하 상태라고 판단되면 갱신을 늦추어 원본부하를 높이지 않는다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <BusySessionCount>100</BusySessionCount>   

-  ``<BusySessionCount> (기본: 100개)``
   원본서버와 HTTP트랜잭션을 진행 중인 세션 수가 일정개수를 넘으면 과부하 상태로 판단한다.
   과부하 상태에서 만료된 컨텐츠를 갱신하기 위해 원본서버로 접속하지 않도록 TTL을 :ref:`caching-policy-ttl` 중 ``<OriginBusy>`` 만큼 연장한다.
   무조건 원본서버로 요청이 가도록 하려면 이 값을 아주 크게 설정하면 된다.
   

.. _origin-balancemode:

원본 선택
====================================

원본서버 주소가 멀티(2개 이상)로 구성되어 있을 때 원본서버 선택정책을 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>RoundRobin</BalanceMode>   

-  ``<BalanceMode> (기본: RoundRobin)``

   -  ``RoundRobin (기본)`` 
      모든 원본서버가 균등하게 요청을 받도록 Round-Robin으로 동작한다. 
      연결된 Idle세션은 해당 서버로 요청이 필요할 때만 사용한다.
   
   -  ``Session``      
      재사용할 수 있는 세션이 있다면 우선 사용한다. 
      신규 세션이 필요하면 Round-Robin으로 할당한다.
      
=========== =================================================================== =====================================================
/           RoundRobin                                                          Session
=========== =================================================================== =====================================================
부하(요청)	모든 서버가 부하를 균등하게 분배	                                반응성과 재사용성이 좋은 서버로 로드가 가중됨
연결비용	높음 (해당 서버의 순서가 되면 연결된 세션을 찾고 없으면 연결시도)   낮음 (재사용할 수 있는 세션이 없을 때만 연결)
재사용성	낮음 (서버 분배 우선)	                                            높음 (항상 연결된 세션을 우선 사용)
세션수	    많음 (각 서버마다 동시에 진행되는 HTTP 트랜잭션의 합)               적음 (동시에 진행되는 HTTP 트랜잭션 만큼 세션 존재)
=========== =================================================================== =====================================================


세션 재사용
====================================

원본서버가 Keep-Alive를 지원한다면 연결된 세션은 항상 재사용된다. 
하지만 세션을 재사용하여 보낸 요청에 대해 원본서버가 일방적으로 연결을 종료할 수 있다.
때문에 연결을 복구하느라 사용자 반응성이 늦어질 가능성이 있다.
특히 오랫동안 재사용하지 않은 세션의 경우 이러한 가능성은 더욱 높다. 
이를 방지하기 위하여 n초 동안 재사용되지 않은 세션에 대해서 자동으로 연결을 종료하도록 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <ReuseTimeout>60</ReuseTimeout>   

-  ``<ReuseTimeout> (기본: 60초)`` 
   일정 시간동안 사용되지 않은 원본세션은 종료한다.
   0으로 설정하면 원본서버 세션을 재사용하지 않는다.
   
   
.. _origin_partsize:

Range요청
====================================

한번에 다운로드 받는 컨텐츠 크기를 설정한다.
동영상처럼 앞 부분만이 주로 소비되는 컨텐츠의 경우 다운로드 크기를 제한하면 불필요한 원본 트래픽를 줄일 수 있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <PartSize>0</PartSize>   

-  ``<PartSize> (기본: 0 MB)``
   0보다 크면 클라이언트가 요청한 지점부터 설정크기(MB) 만큼 Range요청으로 다운로드 한다.   


``<PartSize>`` 를 사용하는 또 다른 이유는 디스크 공간을 절약하기 위함이다.
기본설정으로 STON은 원본크기의 파일을 디스크에 생성한다.
하지만 ``<PartSize>`` 가 0이 아니라면 다운로드 되는만큼만 파일을 분할하여 저장한다.

예를 들어 1시간짜리 영상(600MB)을 1분(10MB)만 시청한 경우에 디스크 공간을 10MB만 사용한다.
공간을 절약하는 장점은 있지만 파일이 분할되어 저장되기 때문에 디스크 부하가 조금 높아진다.

.. note::

   최초 콘텐츠를 다운로드할 때 Content-Length를 알 수 없으므로 Range요청을 할 수 없다. 
   때문에 ``<PartSize>`` 가 설정되어 있다면 설정크기만큼만 다운로드 받고 연결을 종료한다.
   
      


전체 Range 초기화
====================================

일반적으로 원본서버로부터 처음 파일을 다운로드 할 때나 갱신확인 할 때는 다음과 같이 단순한 형태의 GET 요청을 보낸다. ::

    GET /file.dat HTTP/1.1
    
하지만 원본서버가 일반적인 GET요청에 대하여 항상 파일을 변조하도록 설정되어 있다면 원본파일 그대로를 Caching할 수 없어서 문제가 될 수 있다.

가장 대표적인 예는 Apache 웹서버가 mod_h.264_streaming같은 외부모듈과 같이 구동되는 경우이다.
Apache 웹서버는 GET요청에 대해서 항상 mod_h.264_streaming모듈을 통해서 응답한다.
클라이언트(이 경우에는 STON)는 원본파일 그대로가 아닌 모듈에 의해 변조된 파일을 서비스 받는다.

   .. figure:: img/conf_origin_fullrangeinit1.png
      :align: center
      
      mod_h.264_streaming모듈은 항상 원본을 변조한다.

Range요청을 사용하면 모듈을 우회하여 원본을 다운로드할 수 있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <FullRangeInit>OFF</FullRangeInit>   

-  ``<FullRangeInit>``

   - ``OFF (기본)`` 일반적인 HTTP요청을 보낸다.
   
   - ``ON`` 0부터 시작하는 Range요청을 보낸다. 
     Apache의 경우 Range헤더가 명시되면 모듈을 우회한다. ::
      
        GET /file.dat HTTP/1.1
        Range: bytes=0-
    
     최초로 파일 Caching할 때는 컨텐츠의 Range를 알지 못하므로 Full-Range(=0부터 시작하는)를 요청한다. 
     원본서버가 Range요청에 대해 정상적으로 응답(206 OK)하는지 반드시 확인해야 한다.

콘텐츠를 갱신할 때는 다음과 같이 **If-Modified-Since** 헤더가 같이 명시된다.
원본서버가 올바르게 **304 Not Modified** 로 응답해야 한다. ::

   GET /file.dat HTTP/1.1
   Range: bytes=0-
   If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT
    
.. note::

   ``<FullRangeInit>`` 가 정상동작함을 확인한 웹서버들 목록.
    
   - Microsoft-IIS/7.5
   - nginx/1.4.2
   - lighttpd/1.4.32
   - Apache/2.2.22
    

.. _origin-httprequest:
    
원본요청 헤더
====================================

Host 헤더
---------------------

원본서버로 보내는 HTTP요청의 Host헤더를 설정한다.
별도로 설정하지 않은 경우 가상호스트 이름이 명시된다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <Host />   

-  ``<Host>``
   원본서버로 보낼 Host헤더를 설정한다.
   원본서버에서 80포트 이외의 포트로 서비스하고 있다면 반드시 포트 번호를 명시해야 한다. ::
   
      # server.xml - <Server><VHostDefault><OriginOptions>
      # vhosts.xml - <Vhosts><Vhost><OriginOptions>
      
      <Host>www.example2.com:8080</Host>


클라이언트가 보낸 Host헤더를 원본으로 보내고 싶은 경우 *로 설정한다.


User-Agent 헤더
---------------------

원본서버로 보내는 HTTP요청의 User-Agent헤더를 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent>STON</UserAgent>   

-  ``<UserAgent> (기본: STON)``
   원본서버로 보낼 User-Agent헤더를 설정한다.


XFF(X-Forwarded-For) 헤더
---------------------

클라이언트와 원본서버 사이에 STON이 위치하면 원본서버는 클라이언트 IP를 얻을 수 없다.
때문에 STON은 원본서버로 보내는 모든 HTTP요청에 X-Forwarded-For헤더를 명시한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <XFFClientIPOnly>OFF</XFFClientIPOnly>   

-  ``<XFFClientIPOnly>``
   
   - ``OFF (기본)`` 클라이언트(IP: 128.134.9.1)가 보낸 XFF헤더에 클라이언트 IP를 추가한다.
     클라이언트가 XFF헤더를 보내지 않았다면 클라이언트 IP만 명시된다. ::
      
        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1
   
   - ``ON`` XFF헤더의 첫번째 주소만을 원본서버로 전송한다. ::
   
        X-Forwarded-For: 220.61.7.150


ETag 헤더 인식
---------------------

원본서버에서 응답하는 ETag인식여부를 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <OriginalETag>OFF</OriginalETag>   

-  ``<OriginalETag>``
   
   - ``OFF (기본)`` ETag헤더를 무시한다.
   
   - ``ON`` ETag를 인식하며 컨텐츠 갱신시 If-None-Match헤더를 추가한다.

   
Redirect 추적
====================================

원본서버에서 Redirect계열(301, 302, 303, 307)로 응답하는 경우 Location헤더를 추적하여 콘텐츠를 요청한다. 

   .. figure:: img/conf_redirectiontrace.png
      :align: center
      
      클라이언트는 Redirect여부를 모른다.

::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <RedirectionTrace>OFF</RedirectionTrace>   

-  ``<RedirectionTrace>``

   - ``OFF (기본)`` 3xx 응답으로 저장된다.
   
   - ``ON`` Location헤더에 명시된 주소에서 콘텐츠를 다운로드 한다.
     형식에 맞지 않거나 Location헤더가 없는 경우에는 동작하지 않는다.
     무한히 Redirect되는 경우를 방지하기 위하여 1회만 추적한다.

