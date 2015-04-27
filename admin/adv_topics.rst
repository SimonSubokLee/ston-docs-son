.. _adv_topics:

Chapter 14. Optimization and More
******************

This chapter explains a few more about advanced topics.
Optimization is an important vale for high performance and vice versa for enterprise environment.

Memory is the most important resource for configuration. 
The best service quality requires a clear understanding of memory indexing (seeking the requested content).
The following table is the recommended memory size for content caching.

============= ============== =============== ============= ========
Physical RAM  System Free    Contents        Caching Count Sockets
============= ============== =============== ============= ========
1GB           409.60MB       188.37MB        219,469       5,000
2GB           819.20MB       446.74MB        520,494       10,000
4GB           1.60GB         963.49MB        1,122,544     20,000
8GB           3.20GB         2.05GB          2,440,422     20,000
16GB          6.40GB         4.45GB          5,303,733     20,000
32GB          12.80GB        9.25GB          11,030,356    20,000
64GB          25.60GB        18.85GB         22,483,603    20,000
128GB         51.20GB        38.05GB         45,390,095    20,000
============= ============== =============== ============= ========

  

.. toctree::
   :maxdepth: 2
   
   
   
.. _adv_topics_indexing:

Memory Indexing
====================================

'Hot' content should be recognised from cold one. 

.. figure:: img/indexing_hot_cold.png
   :align: center

Content cached from origin servers is stored at local disks of STON Edge Server.
The cache content loading might be slow if read from the disks.
It is a lot faster if read from memory.
Content cached on memory is 'hot', and one on the disk is 'cold.'

Indexing is for seeking cold and hot content for the best performance. Memory indexing is the default. ::

   # server.xml - <Server><Cache>
   
   <Indexing>Memory</Indexing>

Memory indexing does not keep record of cold content. Information about all contents are loaded in memory. The recommended physical memory size is shown in the table above. 

If the requested content is not found among hot contents, then the request is passed onto disk indexing, which keep the record of cold contents. ::

   # server.xml - <Server><Cache>
   
   <Indexing>Disk</Indexing>
   
Virtually an unlimited number of files can be cached by this way. Hot content would be served instantly from memory. However cold content may not be served fast because of slow disk read. The serving speed would be limited to memory read speed for hot ones, and to disk read speed for cold ones.

SSD (Solid-State Drive) is highly recommended if running disk indexing.
Indexing utilizes the disk in which STON is installed. 
Installing STON on SSD helps high performance caching.

.. note::

   SSDs have lifespans, decided by writing.
   SSDs from Intel or Samsung guarantee 600 terabytes of writing at least.
   If 20 gigabytes is written a day, its lifespan would be about 10 years.
   99 percent of writing operations from STON is logging.
   Therefore it is highly recommended to keep logging on other disks than SSDs.


.. warning::

   Indexing mode cannot be changed dynamically (while running service).
   STON must be restarted :ref:`getting-started-reset` after switching the indexing mode.



.. _adv_topics_mem:

Memory Structure
====================================

Cache Server and generic web server have different purposes, although they seem to have much in common.
Understanding structure and mechanism helps optimization.
The purpose of optimization is as follows.

**Massive sesssion handling**. Tens of thousands of simultaneous sessions 

**Instant Response**. Response service for clients

**Origin Off-loading**. Origin overload prevented in advance

The following is a sample segmentation for 8GB and 16GB physical memory.

.. figure:: img/perf_mem_8_16.png
   :align: center

STON shares physical memory, depending on sockets and caching files to serve.  

.. note::

   Disk I/O speed might be a drag the service performance. 
   Deciding caching size for the least disk IO is the primary 
   
   
   
.. _adv_topics_mem_control:

Memory Management
====================================

`Memory Structure`_ is automatically calculated based on physical memory. ::

   # server.xml - <Server><Cache>
   
   <SystemMemoryRatio>100</SystemMemoryRatio>

-  ``<SystemMemoryRatio> (default: 100)`` memory assgined for STON

If ``<SystemMemoryRatio>`` is set to 50 with 8GB of physical memory, STON runs within 4GB memory.
This feature may be useful if other processes run simultaneously. 

Content memory is also adjustable for the best performance. ::

   # server.xml - <Server><Cache>
   
   <ContentMemoryRatio>50</ContentMemoryRatio>

-  ``<ContentMemoryRatio> (default: 50)`` memory segment assgined for content caching (within ``<SystemMemoryRatio>``).

Higher ConntentMemoryRatio means less file IO for serving large sized contents such as game portals.
On the other hand, less ContentMemoryRatio helps serving many and small sized contents.



.. _adv_topics_sys_free_mem:

System Free Memory
====================================

Slow OS(Operating System) is an absoulte drag for any application.
STON leaves some free memory for operating system, which is system free memory.

.. note::

   We have been anticipating for a good reasoning and found
    `this article <http://www.sysxperts.com/home/announce/vmdirtyratioandvmdirtybackgroundratio>`_ .

============== ===============
Physical RAM   System Free
============== ===============
1GB	           409.6MB
2GB	           819.2MB
4GB            1.6GB
8GB	           3.2GB
16GB	         6.4GB
32GB	         12.8GB
64GB	         25.6GB
128GB	         51.2GB
============== ===============

Free memory ratio is adjustable depending on service characteristics. Less free memory means more contents allocated. ::

   # server.xml - <Server><Cache>
   
   <SystemFreeMemoryRatio>40</SystemFreeMemoryRatio>

-  ``<SystemFreeMemoryRatio> (default: 40, maximum: 40)`` Free memory left 물리 메모리를 기준으로 설정된 비율만큼을 Free메모리로 남겨둔다.



Caching Service Memory
====================================

클라이언트에게 전송할 컨텐츠를 Caching하는 메모리이다. 
한번 디스크에서 메모리로 적재된 컨텐츠는 메모리 부족현상이 발생하지 않는다면 계속 메모리에 존재한다. 
문제는 메모리 부족현상은 항상 발생한다는 점이다.

.. figure:: img/perf_inmemory.png
   :align: center

위 그림처럼 전송해야할 컨텐츠는 디스크에 가득한데 실제 메모리에 적재할 수 있는 용량은 아주 제한적이다. 
32GB의 물리 메모리를 장착한다해도 고화질 동영상이나 게임 클라이언트의 크기를 감안한다면 그리 넉넉한 편은 아니다. 
아무리 효율적으로 메모리를 관리해도 물리적인 디스크 I/O속도에 수렴할 수 밖에 없다. 

가장 효과적인 방법은 Contents메모리 공간을 최대한 확보하여 디스크 I/O를 줄이는 것이다. 
다음은 물리 메모리 기준으로 STON이 기본으로 설정하는 최대 Contents메모리 크기이다.

=============== ================= ====================
Physical RAM    Contents          Caching Count
=============== ================= ====================
1GB             188.37MB          219,469
2GB             446.74MB          520,494
4GB             963.49MB          1,122,544
8GB             2.05GB            2,440,422
16GB            4.45GB            5,303,733
32GB            9.25GB            11,030,356
64GB            18.85GB           22,483,603
128GB           38.05GB           45,390,095
=============== ================= ====================



Socket Memory
====================================

소켓도 메모리를 사용한다.
4GB이상의 장비에서 STON은 2만개의 소켓을 기본으로 생성한다. 
소켓 1개=10KB, 1만개당 97.6MB의 메모리를 사용하므로 약 195MB의 메모리가 기본으로 소켓에 할당된다.

=============== ================= ======================
Physical RAM    Socket Count      Socket Memory
=============== ================= ======================
1GB             5천               97.6MB
2GB             1만               195MB
4GB 이상        2만               390MB
=============== ================= ======================

다음 그림처럼 소켓을 모두 사용하면 자동으로 소켓이 늘어난다.
                     
.. figure:: img/perf_sockets.png
   :align: center
    
위 그림과 같이 증설되어 3만개의 소켓을 사용한다면 총 240MB의 메모리가 소켓에 할당된다. 
필요한 소켓을 필요한만큼만 사용하는 것은 아무 문제가 없어 보인다. 
하지만 사용하지 않는 소켓을 지나치게 많이 설정해놓는 것은 메모리 낭비다.
예를 들어 10Gbps장비에서 사용자마다 10Mbps의 전송속도를 보장한다고 가정했을 때 다음 공식에 의하여 최대 동시 사용자는 1,000명이다. ::

   10,000Mbps / 10Mbps = 1,000 Sessions
   
이 경우 STON이 최초 생성하는 2만개 중 19,000개에 해당하는 약 148MB는 낭비가 되는 셈이다.
이 148MB를 Contents에 투자한다면 효율을 더 높일 수 있다. 
최소 소켓수를 설정하면 메모리를 보다 효율적으로 사용할 수 있다.

**최소 소켓수**. 최초에 할당되는 소켓수를 의미한다.

**증설 소켓수**. 소켓이 모두 사용 중(Established)일 때 설정한 개수만큼 소켓을 증설한다.

또 하나의 중요한 변수는 클라이언트 Keep-Alive시간 설정이다. (:ref:`handling_http_requests_session_man` 참조)

.. figure:: img/perf_keepalive.png
   :align: center

연결된 모든 소켓이 데이터 전송 중에 있는 것은 아니다.
IE, Chrome과 같은 브라우저들은 다음에 발생할 HTTP전송을 위해 소켓을 서버에 접속해 놓은 상태로 유지한다.
실제로 쇼핑몰의 경우 연결되어 있는 세션 중 아무런 데이터 전송이 발생하지 않고 그저 붙어 있는 세션의 비율은 적게는 50%에서 많게는 80%에 이른다.

.. figure:: img/perf_keepalive2.png
   :align: center

Keep-Alive시간을 길게 줄수록 소켓의 재사용성은 좋아지지만 유지되는 Idle소켓의 개수가 증가하므로 메모리 낭비가 심해진다.
그러므로 서비스에 맞는 적절한 클라이언트 Keep-Alive시간을 설정하는 것이 중요하다.




Client Access Capping
====================================

제한없이 클라이언트 요청을 모두 허용하면 시스템에 지나친 부하가 발생할 수 있다. 
시스템 과부하는 사실상 장애이다.
적절한 수치에서 클라이언트 요청을 거부하여 시스템을 보호한다. ::

   # server.xml - <Server><Cache>
   
   <MaxSockets Reopen="75">80000</MaxSockets>

-  ``<MaxSockets> (기본: 80000, 최대: 100000)`` 연결을 허용할 최대 클라이언트 소켓 수. 
   이 수치를 넘으면 신규 클라이언트 접속을 즉시 닫는다.
   ``<MaxSockets>`` 의 ``Reopen (기본: 75%)`` 비율만큼 소켓 수가 감소하면 다시 접속을 허용한다.

.. figure:: img/maxsockets.png
   :align: center

(기본 설정에서) 전체 클라이언트 소켓 수가 8만을 넘으면 신규 클라이언트 접속은 즉시 종료된다.
전체 클라이언트 소켓 수가 6만(8만의 75%)이 되면 다시 접근을 허용한다.

예를 들어 3만개의 클라이언트 세션을 처리할 때 원본 서버들이 모두 한계에 도달하면  
이 수치를 3~4만 정도로 설정하는 것이 좋다. 
이로 인해 얻을 수 있는 효과는 다음과 같다.

-  별다른 Network 구성(e.g. L4 세션조절 등)이 필요 없다.
-  불필요한(원본 부하로 처리될 수 없는) 클라이언트 요청을 방지한다.
-  서비스의 신뢰성을 높인다. 서비스 Burst 이후 재시작 등 점검 작업이 필요 없다.



HTTP Client Session
====================================

HTTP 클라이언트 연결을 처리하기 위한 초기/증설 세션 수를 설정한다. ::

    # server.xml - <Server><Cache>
   
    <HttpClientSession>
       <Init>20000</Init>
       <TopUp>6000</TopUp>
    </HttpClientSession>
    
-  ``<Init>`` STON 시작시 미리 생성해놓는 소켓 수

-  ``<TopUp>`` 생성해놓은 소켓 수를 초과했을 때 추가로 생성할 소켓 수

별도로 설정하지 않을 경우 물리 메모리 크기에 따라 자동으로 설정된다.

=============== =========================
물리메모리	    <Init>, <TopUp>
=============== =========================
1GB             5천, 1천
2GB             1만, 2천
4GB             2만, 4천
8GB 이상        2만, 6천
=============== =========================
제한적인 환경에서 적은 수의 소켓만으로도 서비스가 가능할 때 소켓 수를 줄이면 메모리를 절약할 수 있다.



Request Hit Ratio
====================================

First of all, you should understand how HTTP requests from clients are processed.
Cache processing results use TCP_* format just like that of Squid, and each expression stands for the process method.

-  ``TCP_HIT`` The requested resource(not expired) is already cached and will respond immediately.
-  ``TCP_IMS_HIT`` The requested resource with an IMS(If-Modified-Since) header is not expired and is still cached, so respond to the client with 304 NOT MODIFIED. When TTLExtensionBy4xx and TTLExtensionBy5xx are set to ON, this will also respond with 304 NOT MODIFIED.
-  ``TCP_REFRESH_HIT`` The requested resource is expired and needs to check the origin server(origin not modified, 304 NOT MODIFIED) to respond. The resource expiration is extended.
-  ``TCP_REF_FAIL_HIT`` Responds with expired content when confirmation from the origin server during the TCP_REFRESH_HIT process fails due to connection failure or transfer delay.
-  ``TCP_NEGATIVE_HIT`` Responds with a corresponding status when the requested resource is cached in an abnormal status(origin server connection/transfer failure, 4xx response, 5xx response).
-  ``TCP_REDIRECT_HIT`` Responds with Redirect based on the service Allow/Deny/Redirect conditions.
-  ``TCP_MISS`` The requested resource is not cached(a first time request). Respond with the result from the origin server.
-  ``TCP_REF_MISS`` The requested resource is expired so respond after the origin server check(origin modified, 200 OK). The new resource is cached.
-  ``TCP_CLIENT_REFRESH_MISS`` Bypasses the request to the origin server.
-  ``TCP_ERROR`` The requested resource is not cached(a first time request). Origin server failures(connection failure, transfer delay, origin exclusion) interrupt resource caching. Respond to the client with 500 Internal Error.
-  ``TCP_DENIED`` The request is denied.

With the above results, the request hit ratio formula can be expressed, as shown below. ::

   TCP_HIT + TCP_IMS_HIT + TCP_REFRESH_HIT + TCP_REF_FAIL_HIT + TCP_NEGATIVE_HIT + TCP_REDIRECT_HIT
   ------------------------------------------------------------------------------------------------
                                            SUM(TCP_*)
                                            

Byte hit ratio
====================================

The byte hit ratio stands for the ratio of transmitted traffic(Client Outbound) to clients to received traffic(Origin Inbound) from origin servers.
A negative value can be obtained if the origin server traffic is higher than the client traffic. ::

   Client Outbound - Origin Inbound
   --------------------------------
           Client Outbound
           

Origin Server Failure Policy
====================================

STON allows customers to inspect the origin server whenever they want.
When origin server failure is detected, the corresponding server is automatically inactivated and switched to recovery mode. 
Even if the server is reactivated, normal service status has to be confirmed in order to run the service.

If all origin servers are failing, the service will be provided by currently cached content. 
Content with expired TTL will be automatically extended until origin servers are recovered. 
Even purged content can be recovered if it cannot be cached from origin servers for seamless service. 
With this policy, clients should not be exposed to the system fail.
If a new content request is received from the client during total system failure, the following error page will be shown with an explanation.

.. figure:: img/faq_stonerror.jpg
   :align: center
      
   Your clients do not want to see this page.
   
   
Time Units and Expressions
====================================

For items that have a base time in "seconds", a string can be used for time expression. 
The following are supported time expressions and the values are converted into seconds.

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
    
These expressions can be used for the following.

- Time expression of Custom TTL
- Everything but the Ratio of TTL
- ClientKeepAliveSec
- ConnectTimeout
- ReceiveTimeout
- BypassConnectTimeout
- BypassReceiveTimeout
- ReuseTimeout
- Cycle attribute of Recovery
- Bandwidth Throttling



Origin Server Distribution
====================================

When there are billions of content to be serviced, it is impossible and inefficient to cache it all. 
The cache server can be upgraded to cache more content, but this method is not economically effective. 
The most effective method is to disperse content to multiple domains and configure with multiple servers.

.. figure:: img/faq_distdomain.jpg
   :align: center
      
   Multi-domain is advantageous for service expansion.
   
If a service is configured with only one domain like (A), there is no way to physically split traffic. 
Service (B) separates multiple domains and configures separate cache servers to split traffic.
However, having multiple domains could be inappropriate in some cases. 

For example, when traffic is low compared to the size of content or the cost of a service update is too high, having multiple domains is not adequate. 
Distributed cache could be a good alternative in these cases. 
Distributed cache does not modify the origin, but share content from the cache server.

.. figure:: img/faq_dist.jpg
   :align: center
      
   Well-known distributed cache methods.

(C) explains how the L7 load balancer analyzes client requests and distributes them to each cache server based upon appointed rules. 
However, the service could become subordinated to the L7 device, which could be problematic when expanding the service. 
In addition, when multiple resources are requested through a single HTTP session, it is possible to request uncached contents. 

(D) explains a method that shares content among cache servers. 
The content that is missing from #1 will be obtained from #2 and serviced. 
This might seem efficient, but there is a disadvantage. 
The topology becomes very complicated and the internal traffic rises sharply for additional servers. 
Moreover, clients might have to wait until data is brought from another cache server instead of the cache server they are connected to.
For this reason, the service quality could deteriorate.

STON suggests a three-tier structured distributed cache. 

Let's begin with the Child(Edge) server that is directly connected to clients. 
Once HTTP establishes a connection with a server, several HTTP transactions are executed. 
In the case of web pages, DNS query and socket connection take more time than data transfer. 
Clients experience faster service when they are provided all data directly from connected servers. 
Therefore, child servers always have to keep the hottest(most frequently accessed) content. 
This guarantees a fast response. 

The most frequently accessed pages from clients will be cached in the child server. 
Which cache server may the clients access, any pages can be serviced quickly.
Above all, there is no doubt that the child server must cache the hottest content all the time. 

For example, let's say there is a total 20 million items in the origin server and the cache server can cache 10 million. 
In the case of a two-tier structure, the child server sends frequent requests to the origin server in order to get the remaining 10 million items that have not been cached. 
As the service gets bigger, the load on the origin server gets heavier as well. 

To effectively overcome this disadvantage, configure cache servers in between child servers and the origin server. 
This might seem meaningless; however, if clients distribute their requests, it's a whole different story. 

Now, let's install two parent servers between the child servers and the origin server.
Each parent server can cache 10 million items.
All child servers can split their requests based on a hash algorithm; for example, odd numbered content requests go to the left server and even numbered content requests go to the right server. 
This configuration will concentrate most requests to the parent servers.
As a result, all content can be cached in cache servers; simply install additional child servers to configure a less burdened topology.

.. figure:: img/faq_3tierdist.jpg
   :align: center
      
   Distributed Cache of Content.

A distributed cache is configured in the virtual host of the child server. ::

    # server.xml - <Server><VHostDefault><OriginOptions>
    # vhosts.xml - <Vhosts><Vhost><OriginOptions>

    <Distribution>OFF</Distribution>
    
-  ``<Distribution>`` Configures the distribution mode of the origin server.

   - ``OFF (default)`` ``<BalanceMode>`` becomes a determinant.
   
   - ``ON`` splits content requests. ``<BalanceMode>`` is ignored. 

If the origin content increases, you can simply put in an additional parent server. 
Child servers still cache the hottest content first and Long-Tail content is cached via the fastest and most credible path. 
When parent servers are failing, child servers exclude failed servers and redistribute content. 
If there are available standby servers, replace failed parent servers with standby servers so other parent servers are not affected.


Emergency Mode
====================================

STON is designed so that all virtual hosts are sharing MemoryBlock and managing data.
When new memory is needed, STON will reuse the MemoryBlock that has not been accessed for a long time. 
This process is called Memory-Sway.
This architecture guarantees stability during long periods of service.

.. figure:: img/faq_emergency1.png
   :align: center
      
   Content data is loaded on the MemoryBlock and serviced.

The left figure on the above illustration explains a situation when all MemoryBlocks are occupied and no MemoryBlock can be reused. 
Memory-Sway is not available in this situation. 
For example, the worst case happens when all clients are downloading tiny parts of different data portions, and at the same time, the origin server is transmitting tiny sizes of different data. 
Allocating new memory from the system could be a solution; 
however, persistent instances like these will evidently raise memory usage. 
An excessive use of memory will cause a system memory swap, or at the worst, OS could force STON to quit.

.. note::

   "Emergency mode" stands for a situation when there is insufficient memory, during which new MemoryBlock allocation is temporarily prohibited.

Emergency mode protects the service from excessive memory usage, and it will be released when enough reusable MemoryBlocks are secured. ::

    # server.xml - <Server><Cache>
   
    <EmergencyMode>OFF</EmergencyMode>    
    
-  ``<EmergencyMode>``

   - ``OFF (default)`` Do not use Emergency Mode.
   
   - ``ON`` Use Emergency Mode.

In Emergency mode, STON operates as follows:

- Services loaded content normally.
- Bypass works normally.
- Returns unloaded content with 503 service temporarily unavailable. TCP_ERROR status increases.
- Quickly cleans up idle client sockets.
- Does not cache modified content.
- TTL does not renew expired content.
- Returns the cache.vhost.status of SNMP and Host.State value of XML/JSON statistics as "Emergency".
- Info log writes Emergency mode activate/inactivate as shown below. ::

    2013-08-07 21:10:42 [WARNING] Emergency mode activated. (Memory overused: +100.23MB)
    ...(skip)...
    2013-08-07 21:10:43 [NOTICE] Emergency mode inactivated.
    
    
Disk Hot-Swap
====================================

Without stopping the service, you can swap the disks. 
The parameter must be the same as the ``<Disk>`` configuration. ::

   http://127.0.0.1:10040/command/unmount?disk=...
   http://127.0.0.1:10040/command/umount?disk=...

An excluded disk is immediately inactivated and all contents in the corresponding disk becomes invalid. 
The status of the disk that is excluded by the administrator is set to "Unmounted".

In order to reactivate the disk, the following should be called. ::

   http://127.0.0.1:10040/command/mount?disk=...

All content in the reactivated disk will become invalid.
