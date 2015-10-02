.. _bypass:

Chapter 7. Bypass
******************

This chapter explains how to set a bypass that will delegate client request handling to the origin server.
Bypassing is distinguished from caching by the conditions and reactions thereof.

Bypass has priority over the caching policy.
If a service did not adopt the Edge during its design stage, it most likely cannot distinguish between static resources and dynamic resources.
In this case, configure to bypass all client requests and only cache the content that is frequently requested according to the log.
Usually a few hours of logging can dramatically decrease the origin server load.
:ref:`monitoring_stats` provides real time status in order to help tune the service in real time.

A bypass is fast and acts as an HTTP transactions basis.
Even if the web site is personalized, mostly just the main page(.html) changes dynamically and the remaining of 99% is made up of static resources.
A bypass version of :ref:`origin-httprequest` is provided in order to make the origin request header be identified by the origin server.


.. toctree::
   :maxdepth: 2


No-Cache Request Bypass
====================================

If a client sends a no-cache request, bypass the request. ::

   GET / HTTP/1.1
   cache-control: no-cache or cache-control:max-age=0
   pragma: no-cache
    
::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BypassNoCacheRequest>OFF</BypassNoCacheRequest>   
    
-  ``<BypassNoCacheRequest>``

   - ``OFF (default)`` The cache module handles the request.
   
   - ``ON`` Bypasses the request to the origin server.
   
.. note::

    This configuration is judged by the client's action(probably ``ctrl`` + ``F5`` ).
    Therefore, an excessive number of bypasses might increase origin load.
    

.. _bypass-getpost:    
 
GET/POST Bypass
====================================

A bypass can be set as a default reaction for GET/POST requests.
The default reaction of the two requests might be different as GET and POST have different usages. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BypassPostRequest>ON</BypassPostRequest>
   <BypassGetRequest>OFF</BypassGetRequest>   
    
-  ``<BypassPostRequest>``

   - ``ON (default)`` Bypasses POST requests to the origin server.
   
   - ``OFF`` STON handles POST requests.
   
-  ``<BypassGetRequest>``

   - ``OFF (default)`` STON handles GET requests.

   - ``ON`` Bypasses GET requests to the origin server.

A bypass supports all conditions as :ref:`access-control-vhost_acl`.
An exception for a bypass is saved at /svc/{virtual host name}/bypass.txt. ::

   # /svc/www.example.com/bypass.txt
   $IP[192.168.2.1-255]
   /index.html   
    
If a cache or bypass condition has not been specified, the opposite setting of the default condition will be applied.
For example, if ``<BypassGetRequest>`` is set to ``ON``, the exceptional condition is a caching list.
If this is confusing, you can use a second parameter to make the condition more clear. ::

   # /svc/www.winesoft.co.kr/bypass.txt
   
   $HEADER[cookie: *ILLEGAL*], cache               // Always cache
   !HEADER[referer:]                               // Depends on the default setting
   !HEADER[referer] & !HEADER[user-agent], bypass  // Always bypass
   $URL[/source/public.zip]                        // Depends on the default setting

The priority of actions is listed below:

1. No-Cache bypass
2. Bypass is stated in bypass.txt
3. Default setting of bypass.txt
    


Origin Server Affinity
====================================

Some transactions have to be directly made between the origin server and the client such as login status.
The origin server affinity can be configured in the `GET/POST Bypass`_ property. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassPostRequest OriginAffinity="ON">...</BypassPostRequest>
   <BypassGetRequest OriginAffinity="ON">...</BypassGetRequest>   

-  ``OriginAffinity``

   - ``ON (default)`` This setting guarantees that client requests will be bypassed to an identical server. 
     However, it might not be an identical socket. 
     
     The origin server and all connected sockets may lose connection.
     In this case, new socket connections will be requested from the relative server.
     
     .. figure:: img/private_bypass3.jpg
        :align: center
      
        Requests are always bypassed to the same server.
     
     If a bypassed origin server is inactive due to errors or exclusion from DNS, another server will be used for the bypass.
   
   - ``OFF`` This setting does not guarantee which server will be used for client requests.
   
     .. figure:: img/private_bypass1.jpg
        :align: center
      
        Abides by :ref:`origin-balancemode`.
        


Origin Session Affinity
====================================

Each client socket uses a 1:1 bypass session with the origin server. 

.. figure:: img/private_bypass2.jpg
   :align: center
      
   The client owns the origin session.

The origin session can be fixed with the `GET/POST Bypass`_ property. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BypassPostRequest Private="OFF">...</BypassPostRequest>
   <BypassGetRequest Private="OFF">...</BypassGetRequest>   
    
-  ``Private``

   - ``OFF (default)`` This setting allows for a client session to use a dedicated origin server session.
     Requests are always bypassed to the same server.
     Either the client or the origin server can terminate the session, at which point Other parties' sessions will also be closed.
        
   - ``OFF`` This setting does not use a dedicated session.

Just like the origin server keeps client's login information based on the session, 
it is useful if the system is set up so that client requests must be handled by/within the same socket.

.. note::

   If too many requests are bypassed with ``Private``, the origin server could be loaded with the corresponding number of clients.  
   Also, these origin sessions are owned by clients and this could endanger the server to malicious attacks.
   

Timeout
-----------------------

A bypass usually responds with a result that is dynamically processed from the origin server.
Therefore, the processing speed is slower than static content.
Setting a ``Timeout`` specifically for bypasses is recommended in order to prevent hastily assuming an error condition. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <BypassConnectTimeout>5</BypassConnectTimeout>
   <BypassReceiveTimeout>300</BypassReceiveTimeout>   

-  ``<BypassConnectTimeout> (default: 5 seconds)``   
   If the bypass connection with origin server is not established for the set amount of time, it is handled as a connection timeout.


-  ``<BypassReceiveTimeout> (default: 5 seconds)``
   If the origin server does not respond for the set amount of time during a bypass, it is handled as a reception timeout.
   
   

Bypass Header
====================================

A bypass header configures whether or not to apply the bypass setting of :ref:`origin-httprequest`. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <UserAgent Bypass="OFF">...</UserAgent>
   <Host Bypass="ON"/>
   <XFFClientIPOnly Bypass="ON">...</XFFClientIPOnly>   
    
-  ``Bypass`` Property

   - ``ON`` Specifies a configured header.
        
   - ``OFF`` Specifies relative headers for clients.


.. _bypass-port:

Port Bypass
====================================

A port bypass bypasses all packets of a specific TCP port to the origin server.
This setting is exclusive to virtual hosts. ::

   # vhosts.xml - <Vhosts>
      
   <Vhost Name="www.example">
      <PortBypass>443</PortBypass>
      <PortBypass Dest=”1935”>1935</PortBypass>
   </Vhost>

-  ``<PortBypass>``   
   Bypasses all packets from a designated port to the same port of the origin server.
   The ``Dest`` property will configure the origin server port.

For example,if you are bypassing a 443 port, the result is similar to establishing a direct SSL connection between the client and the origin server. 
The port that is being bypassed can never have a redundant setting. 

.. note::
   
   Structurally, a port bypass is handled in a TCP Layer that is lower than the HTTP. 
   The reason for configuring a port bypass under a specific virtual host is to use the virtual host as a subject when collecting a statistics.

