.. _adv_topics:

Chapter 14. Advanced Topics
******************

This chapter will explain a handful of advanced topics.
Some of the content is bound up with internal structure to assist advanced administrators.

.. toctree::
   :maxdepth: 2



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

Without stopping the service, you can change the disk. 
The parameter must be the same as the ``<Disk>`` configuration. ::

   http://127.0.0.1:10040/command/unmount?disk=...
   http://127.0.0.1:10040/command/umount?disk=...

An excluded disk is immediately inactivated and all contents in the corresponding disk becomes invalid. 
The status of the disk that is excluded by the administrator is set to "Unmounted".

In order to reactivate the disk, the following should be called. ::

   http://127.0.0.1:10040/command/mount?disk=...

All content in the reactivated disk will become invalid.
