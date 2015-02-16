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
-  ``TCP_IMS_HIT`` The requested resource with IMS(If-Modified-Since) header is not expired and still cached, so 304 NOT MODIFIED is responded. When TTLExtensionBy4xx and TTLExtensionBy5xx are set to ON, it'll also return 304 NOT MODIFIED.
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



Origin Server Distribution
====================================

When there are billions of contents to be serviced, it is impossible and inefficient to cache all contents. 
Cache server can be upgraded to cache more contents, but this method is not economically effective. 
The most effective method is to disperse contents domain and configure with multiple servers.

.. figure:: img/faq_distdomain.jpg
   :align: center
      
   Multi-domain is advantageous to service expansion.
   
If a service is configured with only one domain like (A), there is no way to physically split traffics. 
The service (B) separates multiple domains and configures separate cache servers to split traffics.
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
Moreover, clients might have to wait until data is brought from another cache server instead of connected cache server.
For this reason, service quality could be deteriorated.

STON suggests 3 Tier structure distributed cache. 

Let's begin with the Child(Edge) server that is directly connected to clients. 
Once HTTP establishes a connection with a server, several HTTP transactions are executed. 
In case of web pages, DNS query and socket connection take more time than data transfer. 
Clients experience fast service when they are provided all data directly from connected servers. 
Therefore, child server always have to keep the hottest(most frequently accessed) contents. 
This guarantees fast response. 

The most frequently accessed pages from clients will be cached in the child server 
and which cache server clients may connect, pages can be serviced quickly.
Above all, there is no doubt that the child server must cache hottest contents all the time. 

For example, let's say there are total 20 million contents in the origin server and the cache server can cache 10 million contents. 
In case of 2 tier structure, the child server sends frequent requests to the origin server in order to get the rest 10 million contents that have not been cached. 
As the service gets bigger, the load on the origin server gets heavier as well. 

To effectively overcome this disadvantage, configuring cache servers in between child servers and the origin server. 
This might seem meaningless, however, if clients distribute their requests, it's a whole different story. 

Now, let's install two parent servers between child servers and the origin server.
Each parent server can cache 10 million contents.
All child servers can split their requests based on hash algorithm, for example, odd numbered contents request to the left server and even numbered contents request to the right server. 
This configuration will concentrate most requests to parent servers.
As a result, all contents can be cached in cache servers and simply install additional child servers to configure less burdened topology.

.. figure:: img/faq_3tierdist.jpg
   :align: center
      
   Distributed Cache of Contents

Distributed cache is configured in the virtual host of the child server. ::

    # server.xml - <Server><VHostDefault><OriginOptions>
    # vhosts.xml - <Vhosts><Vhost><OriginOptions>

    <Distribution>OFF</Distribution>
    
-  ``<Distribution>`` configures distribution mode of the origin server.

   - ``OFF (default)`` ``<BalanceMode>`` becomes a determinant.
   
   - ``ON`` splits contents requests. ``<BalanceMode>`` is ignored. 

If the origin contents increase, you can simply put additional parent server. 
Child servers still cache hottest contents first and Long-Tail contents are cached via fastest and most credible path. 
When parent servers are failing, child servers exclude failed servers and redistribute contents. 
If there are available standby servers, replace failed parent servers with standby servers so other parent servers are not affected.


Emergency Mode
====================================

The STON is designed that all virtual hosts are sharing MomoeryBlock and managing data.
When a new memory is needed, STON will reuse the MemoryBlock that has not been accessed for a long time to secure new memory. 
This process is called Memory-Sway.
This architecture guarantees stability during long period of service.

.. figure:: img/faq_emergency1.png
   :align: center
      
   Contents data are loaded on the MemoryBlock and serviced.

The left figure of the above illustration explains a situation when all MemoryBlocks are accupied and no MemoryBlock can be reused. 
Memory-Sway is not available in this situation. 
For example, the worst case happens when all clients are downloading tiny parts of different data portions, and at the same time the origin server trasmits tiny size of different data. 
Allocating a new memory from the system could be a solution for the worst case. 
However, persistent worst cases will evidently raise memory usage. 
An excessive use of memory will occur system memory swap, or at the worst, OS could force quit the STON.

.. note::

   Emergency mode stands for a situation when there is insufficient memory and temporarily prohibits new MemoryBlock allocation.

Emergency mode is to protect the service from excessive memory usage, and it will be released when enough size of reusable MemoryBlocks are secured. ::

    # server.xml - <Server><Cache>
   
    <EmergencyMode>OFF</EmergencyMode>    
    
-  ``<EmergencyMode>``

   - ``OFF (default)`` Do not use Emergency Mode.
   
   - ``ON`` Use Emergency Mode.

In Emergency mode, STON operates as below.

- Loaded contents are serviced normally.
- Bypass works normally.
- Unloaded contents will be returned with 503 service temporarily unavailable. TCP_ERROR status increases.
- Quickly cleans up idle client sockets.
- Does not cache modified contents.
- TTL does not renew expired contents.
- The cache.vhost.status of SNMP and Host.State value of XML/JSON statistics is returned as "Emergency".
- Info log writes Emergency mode activate/inactivate as below. ::

    2013-08-07 21:10:42 [WARNING] Emergency mode activated. (Memory overused: +100.23MB)
    ...(skip)...
    2013-08-07 21:10:43 [NOTICE] Emergency mode inactivated.
    
    
Disk Hot-Sawp
====================================

Without stopping the service, you can change the disk. 
The parameter must be the same with ``<Disk>`` configuration. ::

   http://127.0.0.1:10040/command/unmount?disk=...
   http://127.0.0.1:10040/command/umount?disk=...

An excluded disk is immediately inactivated and all contents in the corresponding disk becomes invalid. 
The status of excluded disk by the administrator is set to "Unmounted".

In order to reactivate the disk, the following should be called. ::

   http://127.0.0.1:10040/command/mount?disk=...

All contents in the reactivated disk becomes invalid.
