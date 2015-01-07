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

Origin Address Use Policy
====================================

The usage of Origin address(IP) is determined according to the following elements.

-  :ref:`env-vhost-activeorigin` Address format(IP or Domain) and stanby address
-  `Error Dectection and Recovery`_
-  `Health-Checker`_

The origin address is frequently excluded/recovered when running a service.
STON uses IP table based origin address, and information be accessed by `origin-status`_ API.

If you set the origin address as an IP, it will be simple.
원본주소를 IP로 설정한 경우 (무엇이 매우 간단한지??)매우 간단하다. 

-  Nothing will modify IP list unless you change the configuration.
-  IP address will not be expired by TTL.
-  Exclusion/recovery works based on IP address.

If the origin address is set as a domain, you have to use Resolving to acquire IP.
( recorded in :ref:`admin-log-dns` .)
IP list can be changed dynamically, and all IPs are only valid for valid TTL.

-  Domain is periodically Resolving(1~10 seconds).
-  Organize IP table to use from Resolving result.
-  All IPs are valid as long as TTL is valid, and IPs will not be used if TTL is expired.
-  If an identical IP is Revolving, renew TTL.
-  IP table should not be cleared. (Even if TTL is expired) Latest IPs are not discarded.

Even if you set the origin address as a domain, error and recovery works based on IP address.
원본주소를 Domain으로 설정하여도 장애/복구는 IP기반으로 동작한다. 
(??이 부분은 구체적인 설명이 필요할 것 같습니다. 무슨 내용인지 이해가 되지 않아요 ㅠㅠ)
여기서 미묘한 점이 있다.
DNS 클라이언트(=STON)는 Domain의 모든 IP 목록을 정확히 알 수 없다. 
하지만 사용할 수 없는 IP들만으로 Domain을 구성할 경우 장애상태가 지속될 수 없다.

Error/Recovery policies of the domain address are listed in the below.

-  If all known IP addresses for a domain are inactivated, related domain address will also be inactive.
-  If a new IP is resolved to the inactive domain, the IP will be inactivated.
-  If TTL of all IPs are expired, inactive domain will not be reactivated.
-  At least one of IP addresses of the inactive domain should be recovered in order to reactivate the domain.

It could be quite difficult to understand, but `origin-status`_ API will help you to understand more about the service operation status.



.. _origin-status:

Origin Status Monitoring
====================================

API is used to monitor the origin status of vitual host. ::

   http://127.0.0.1:10040/monitoring/origin       // All virtual hosts
   http://127.0.0.1:10040/monitoring/origin?vhost=www.example.com
   
The result will be returned in JSON format. ::

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
    
-  ``VirtualHost`` Virtual host name

-  ``Address`` :ref:`env-vhost-activeorigin` .
   ``Active`` will be returned if configured address is in use, and ``Inactive`` will be returned when the address is not in use due to an error.

-  ``Address2`` :ref:`env-vhost-standbyorigin` .
   ``Active`` will be returned if configured address is in use, otherwise ``Inactive`` will be returned.

-  ``ActiveIP`` IP list and TTL that are in use. 
   If the origin server is set by IP address, an identical IP of ``Address`` with TTL value of 0 will be returned.
   If it is set by domain, the return value depends on the Resolving result.
   Various IP and TTL are used.
   (누가??)다양한 IP와 TTL을 사용한다.
   
-  ``InactiveIP`` IP list and TTL that are not in use.
   Even though the IP is not in use, it could be in a recovery status or be managed by HealthChecker.
   If the address is not recovered wihtin TTL, it'll be removed.
   

    
.. _origin-status-reset:

Origin Status Reset
====================================

API is used to reset the origin server exclusion/recovery of virtual host. 
Also, current session will not be reused, but a new connection will be created instead. ::

   http://127.0.0.1:10040/command/resetorigin       // All virtual hosts
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com   



.. _origin-busysessioncount:

Overload Judgement
====================================

The contents that are requested for the first time, they must always request to the origin server.
On the other hand, if the contents are already cached, the request can be more flexibly responded.
If the server is already overloaded, renewal is postponed to keep low origin load. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <BusySessionCount>100</BusySessionCount>   

-  ``<BusySessionCount> (default: 100)``
   If the number of HTTP transaction in progress with the origin server exceeds certain a number, it will be judged as a overload.
   In order to block any contents renewal requests to the origin server during the overload status, extend TTL for ``<OriginBusy>`` from :ref:`caching-policy-ttl`.
   You can set a very large value for this option to forward all requests unconditionally to the origin server.
   

.. _origin-balancemode:

Origin Selection
====================================

When the origin server consists of multiple addresses(at least 2 addresses), configure the origin sever selection policy. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>RoundRobin</BalanceMode>   

-  ``<BalanceMode> (default: RoundRobin)``

   -  ``RoundRobin (default)`` 
      Round-Robin is applied so that all origin servers can equally get requests.
      Connected idle session is only used when a request is need to the related server.
   
   -  ``Session``      
      If there is any reusable sessions, make use of them. 
      If new session is required, use Round-Robin to allocate session to the server.
      
=========== =================================================================== =====================================================
/           RoundRobin                                                          Session
=========== =================================================================== =====================================================
Load(Request)  All servers equally get the load	                                Highly reactive and reusable server gets loaded
Connection Cost   High (If connected session is not found in current server, establish new connection)   Low (Connect only when reusable session is not exist)
Reusability	Low (Priority to server distribution)	                                            High (Priority to connected session)
# of Session	    Many (Sum of concurrent HTTP transactions on each server)               Few (There are as many sessions as concurrent HTTP transactions)
=========== =================================================================== =====================================================


Session Recycle
====================================

If the origin server supports Keep-Alive, connected sessions are always recycled.
However, the origin server can unilaterally close the connection for the request from recycled session.
Therefore, reestablishing the connection might cause a delay in user reactivity.
The session that has not been reused for a long time, especially, have more possibility for reactivity issue.
The following configuration will close the connection of unrecycled session for n seconds. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <ReuseTimeout>60</ReuseTimeout>   

-  ``<ReuseTimeout> (default: 60 seconds)`` 
   Close an origin session that has not been used for the set amount time.
   Setting this value to 0 will not reuse the origin server session.
   
   
.. _origin_partsize:

Range Request
====================================

Configure the size of contents to download.
The contents like video clips that are usually consumed from the head of file, unnecessary origin traffic can be reduced by restricting download size. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <PartSize>0</PartSize>   

-  ``<PartSize> (default: 0 MB)``
   If this value is greater than 0, configured size will be downloaded with using Range request from the location where client requested.   


``<PartSize>`` can also help saving disk space.
Ston generates a same-sized file to the original file in the disk.
If ``<PartSize>`` is not 0, downloaded files will be partitioned and saved.

For example, if a client watches first 1 minute(10MB) of the 1 hour(600MB) video clip, only 10MB of disk space is used.
This option will save disk space, but partitioning a file will increase disk load a bit.

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

