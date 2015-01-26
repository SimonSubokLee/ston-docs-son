.. _adv_topics:

Chapter 14. Advanced Topics
******************

This chapter will explain handful advanced topics.
Some of the contents are bounded up with internal structure to assist advanced administrators.

.. toctree::
   :maxdepth: 2



Request Hit Ratio
====================================

First of all, you should understand how HTTP requests from clients are processed.
Cache processing results are using TCP_* format just like that of Squid, and each expression stands for the process method.

-  ``TCP_HIT`` The requested resource(not expired) is already cached and responded immediately.
-  ``TCP_IMS_HIT`` The requested resource with IMS(If-Modified-Since) header is not expired and still cached, so 304 NOT MODIFIED is responded. TTLExtensionBy4xx, TTLExtensionBy5xx설정에 해당하는 경우(TTLExtensionBy4xx/5xx의 어떤 설정에 어떻게 해당되어야 304 NOT MODIFIED 응답이 나오는지??)에도 이에 해당함.
-  ``TCP_REFRESH_HIT`` The requested resource is expired and needs to check the origin server(origin not modified, 304 NOT MODIFIED) to respond. Resource expiration is extended.
-  ``TCP_REF_FAIL_HIT`` When the confirmation from the origin server during TCP_REFRESH_HIT process fails due to connection failure or transfer delay, respond with expired contents.
-  ``TCP_NEGATIVE_HIT`` The requested resource is cached in an abnormal status(origin server connection/transfer failure, 4xx response, 5xx response), then respond with corresponding status.
-  ``TCP_REDIRECT_HIT`` Respond with Redirect based on the service Allow/Deny/Redirect conditions.
-  ``TCP_MISS`` The requested resource is not cached(first time request). Respond with the result from the origin server.
-  ``TCP_REF_MISS`` The requested resource is expired so respond after origin server check(origin modified, 200 OK). The new resource is cached.
-  ``TCP_CLIENT_REFRESH_MISS`` Bypass the request to the origin server.
-  ``TCP_ERROR`` The requested resource is not cached(first time request). Origin server failures(connection failure, transfer delay, origin exclusion) interrupt resource caching. Respond to the client with 500 Internal Error.
-  ``TCP_DENIED`` The request is denied.

With the above results, request hit ratio formula can be expressed as below. ::

   TCP_HIT + TCP_IMS_HIT + TCP_REFRESH_HIT + TCP_REF_FAIL_HIT + TCP_NEGATIVE_HIT + TCP_REDIRECT_HIT
   ------------------------------------------------------------------------------------------------
                                            SUM(TCP_*)
                                            

Byte hit ratio
====================================

The byte hit ratio stands for the ratio of transmitted traffic(Client Outbound) to clients to received traffic(Origin Inbound) from origin servers.
Negative value can be obtained if the origin server traffic is higher than the client traffic. ::

   Client Outbound - Origin Inbound
   --------------------------------
           Client Outbound
           

Origin Server Failure Policy
====================================

The STON allows customers to inspect the origin server whenever they want.
When the origin server failure is detected, corresponding server is automatically inactivated and switched to recovery mode. 
Even if the server is reactivated, normal service status has to be confirmed in order to run the service.

If all origin servers are failing, the service will be provided with currently cached contents. 
Contents with expired TTL will be automatically extended until origin servers are recovered. 
Even purged contents can be recovered if they cannot be cached from origin servers for seamless service. 
With this policy, clients should not be exposed to the failure situation.
A new contents request is received from the client during the total failure, the following error page will be shown with the explanation.

.. figure:: img/faq_stonerror.jpg
   :align: center
      
   Your clients don't want to see this page.
   
   
Time Units and Expressions
====================================

For the items that have base time in "second", a string can be used for time expression. 
The followings are supported time expressions and converted value in second.

=========================== =========================
Expressions	                    Converted Value
=========================== =========================
year(s)                     31536000 sec (=365 days)
month(s)                    2592000 sec (=30 days)
week(s)                     604800 sec (=7 days)
day(s)                      86400 sec (=24 hours)
hour(s)	                    3600 sec (=60 mins)
minute(s), min(s)	        60 sec
second(s), sec(s), (Omitted)	1 sec
=========================== =========================

Combined expression is also supported. ::

    1year 3months 2weeks 4days 7hours 10mins 36secs
    
These expressions can be used for followings.

- Time expression of Custom TTL
- Everything but Ratio of TTL
- ClientKeepAliveSec
- ConnectTimeout
- ReceiveTimeout
- BypassConnectTimeout
- BypassReceiveTimeout
- ReuseTimeout
- Cycle attribute of Recovery
- Bandwidth Throttling



Origin Server Dispersion
====================================

When there are billions of contents to be serviced, it is impossible and inefficient to cache all contents. 
Cache server can be upgraded to cache more contents, but this method is not economically effective. 
The most effective method is to disperse contents domain and configure with multiple servers.

.. figure:: img/faq_distdomain.jpg
   :align: center
      
   Multi-domain is advantageous to service expansion.
   
If a service is configured with only one domain like (A), there is no way to physically disperse traffics. 
The service (B) separates multiple domains and configures separate cache servers to disperse traffic.
However, having multiple domains could be inappropriate in some cases. 

For example, when the traffic is low compare to the size of contents or service update cost is too high, having multiple domain is not adequate. 
Distributed cache could be a good alternative in these cases. 
Distributed cache does not modify the origin, but sharing contents from the cache server.

.. figure:: img/faq_dist.jpg
   :align: center
      
   Well known distributed cache method

(C) explains how the L7 load balancer analyzes client requests and distribute them to each cache server with appointed rules. 
However, the service could be subordinated to L7 device and become problematic when expanding service. 
In addition, when multiple resources are requested through a single HTTP session, there might be a change to request uncached contents. 

(D) explains a method that shares contents among cache servers. 
The contents that are missing from #1 will be obtained from #2 and serviced. 
This might seem efficient, but there is a disadvantage. 
The topology becomes very complicated and the internal traffic sharply rises for additional servers. 
또한 사용자들은 연결된 캐시서버에서 즉각적으로 서비스받지 못하고 다른 캐시서버로부터 
데이터를 가져올 때까지 기다려야 하므로 서비스품질이 저하된다.

우리가 제안하는 분산캐시는 3 Tier구조의 분산캐시이다. 

우선 클라이언트와 직접 연결되는 Child(=Edge)서버부터 이야기를 시작해 보자. 
HTTP는 한번 연결을 맺은 서버와 여러번의 HTTP 트랜잭션을 수행하는 특성을 가진다. 
웹 페이지의 경우 데이터 전송보다 DNS Query와 소켓 연결에 더 많은 시간이 소요된다. 
클라이언트는 자신이 연결된 서버로부터 모든 데이터를 제공받을 때 가장 빠르다. 
결국 Child는 항상 가장 Hot한(=많이 접근되는) 컨텐츠를 캐싱하고 있어야 한다. 
이것이 빠른 반응성을 보장하는 방법이다. 

사용자들이 가장 많이 접근하는 페이지들은 대부분 Child에 캐싱되어 있을 것이며, 
어느 캐시서버에 접속하더라도 해당 페이지를 빠르게 서비스받을 수 있다.
우선 Child는 Hot한 컨텐츠를 항상 캐싱하고 있어야 한다는데는 이견이 없다. 

예를 들어 원본서버의 전체 컨텐츠가 2,000만개이고 한대의 캐시서버가 1,000만개의 컨텐츠를 
캐싱할 수 있다고 가정해보자. 
2 Tier구성일 경우 Child서버는 캐싱하지 못한 1,000만개를 캐싱하기 위해 
항상 원본서버로 요청을 보낸다. 
서비스가 커질수록 원본서버가 많은 부하를 받게 된다. 

이런 단점을 극복하기 위해 Child와 원본서버 사이에 캐시서버를 두면 효과적이다. 
언뜻 큰 의미가 없어 보이기도 한다.
하지만 클라이언트가 요청을 분산해서 보내면 이야기가 달라진다. 

Child와 원본서버 사이에 Parent를 2대 투입한다. 
Parent한대당 1,000만개를 캐싱할 수 있다. 
모든 Chlid들은 해쉬 알고리즘에 의해서 홀수 컨텐츠는 왼쪽 서버에, 
짝수 컨텐츠는 오른쪽 서버로 요청할 수 있다. 
이렇게 설정하면 Parent캐시서버의 집중도가 매우 높아지는 효과가 발생한다. 
결국 원본서버의 부하없이 모든 컨텐츠를 캐시서버팜 안에 캐싱할 수 있을 뿐만 아니라 
간단히 Child서버를 증설하여 부담없는 Topology를 구성할 수 있다.

.. figure:: img/faq_3tierdist.jpg
   :align: center
      
   콘텐츠 분산캐시

분산캐시는 Child서버의 가상호스트에 설정한다. ::

    # server.xml - <Server><VHostDefault><OriginOptions>
    # vhosts.xml - <Vhosts><Vhost><OriginOptions>

    <Distribution>OFF</Distribution>
    
-  ``<Distribution>`` 원본서버 분산모드를 설정한다.

   - ``OFF (기본)`` ``<BalanceMode>`` 에 따라 동작한다.
   
   - ``ON`` 콘텐츠를 분산하여 요청한다. ``<BalanceMode>`` 는 무시된다. 

원본 컨텐츠가 더 늘어나면 Parent서버를 한대 더 투입만 하면된다. 
여전히 Child는 Hot 컨텐츠 위주로 캐싱하며 가장 빠르고 신뢰할 수 있는 경로로 
Long-Tail컨텐츠를 캐싱할 수 있다. 
Parent서버에 장애가 발생하면 Child들은 장애서버를 배제하고 컨텐츠를 다시 분산한다. 
Standby서버가 있다면 장애서버 위치에 Standby서버를 위치시켜 다른 Parent서버가 
영향받지 않게 한다.


Emergency 모드
====================================

내부적으로 모든 가상호스트가 MemoryBlock을 공유하면서 데이터를 관리하도록 설계되어 있다. 
신규 메모리가 필요한 경우 참조되지 않는 오래된 MemoryBlock을 재사용하여 신규 메모리를 확보한다. 
이 과정을 Memory-Swap이라고 부른다. 
이런 구조를 통해 장기간 운영하여도 안정성을 확보할 수 있다.

.. figure:: img/faq_emergency1.png
   :align: center
      
   콘텐츠 데이터는 MemoryBlock에 담겨 서비스된다.

위 그림의 우측 상황처럼 모든 MemoryBlock이 사용 중이어서 재사용할 수 있는 MemoryBlock이 
존재하지 않는 상황이 발생할 수 있다. 
이때는 Memory-Swap이 불가능해진다. 
예를 들어 모든 클라이언트가 서로 다른 데이터 영역을 아주 조금씩 다운로드 받거나 
원본서버에서 서로 다른 데이터를 아주 조금씩 전송하는 상황이 동시에 발생하는 경우가 최악이다. 
이런 경우 시스템으로부터 새로운 메모리를 할당받아 사용하는 것도 방법이다. 
하지만 이런 상황이 지속될 경우 메모리 사용량이 높아진다. 
메모리 사용량이 과도하게 높아질 경우 시스템 메모리스왑을 발생시키거나 최악의 경우 
OS가 STON을 종료시키는 상황이 발생할 수 있다.

.. note::

   Emergency 모드란 메모리 부족상황이 발생할 경우 임시적으로 신규 MemoryBlock의 할당을 금지시키는 상황을 의미한다.

이는 과다 메모리 사용으로부터 스스로를 보호하기 위한 방법이며, 
재사용가능한 MemoryBlock이 충분히 확보되면 자동 해지된다. ::

    # server.xml - <Server><Cache>
   
    <EmergencyMode>OFF</EmergencyMode>    
    
-  ``<EmergencyMode>``

   - ``OFF (기본)`` 사용하지 않는다.
   
   - ``ON`` 사용한다.

Emergency모드일 때 STON은 다음과 같이 동작합니다.

- 이미 로딩되어 있는 컨텐츠는 정상적으로 서비스된다.
- 바이패스는 정상적으로 이루어진다.
- 로딩되어 있지 않은 컨텐츠에 대해서는 503 service temporarily unavailable로 응답한다. TCP_ERROR상태가 증가한다.
- Idle 클라이언트 소켓을 빠르게 정리한다.
- 신규 컨텐츠를 캐싱할 수 없다.
- TTL이 만료된 컨텐츠를 갱신하지 않는다.
- SNMP의 cache.vhost.status와 XML/JSON통계의 Host.State 값이 "Emergency"로 제공된다.
- Info로그에 Emergency모드로 전환/해제를 다음과 같이 기록한다. ::

    2013-08-07 21:10:42 [WARNING] Emergency mode activated. (Memory overused: +100.23MB)
    ...(생략)...
    2013-08-07 21:10:43 [NOTICE] Emergency mode inactivated.
    
    
디스크 Hot-Sawp
====================================

서비스 중단없이 디스크를 교체한다. 
파라미터는 반드시 ``<Disk>`` 설정과 같아야 한다. ::

   http://127.0.0.1:10040/command/unmount?disk=...
   http://127.0.0.1:10040/command/umount?disk=...

배제된 디스크는 즉시 사용되지 않으며 해당 디스크에 저장되었던 모든 컨텐츠는 무효화된다. 
관리자에 의해 배제된 디스크의 상태는 "Unmounted"로 설정된다.

디스크를 서비스에 재투입하려면 다음과 같이 호출한다. ::

   http://127.0.0.1:10040/command/mount?disk=...

재투입된 디스크의 모든 콘텐츠는 무효화된다.
