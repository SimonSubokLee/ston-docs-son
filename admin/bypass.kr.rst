.. _bypass:

Chapter 8. Bypass
******************

This chapter explains how to set a bypass that is delegating client request handling to the origin server.
Bypass is distinguished from caching with conditions and reactions thereof.

Bypass has a priority to the Caching policy.
A service that was not considered to adopt the Edge from design stage, most likely it cannot delicately distinguish static resources and dynamic resources.
In this case, configure to bypass all client requests and only cache the contents that are frequently requested based on the log.
Usually a few hours of log can dramatically decrease the origin server load.
:ref:`monitoring_stats` provides real time status in order to help tuning the service in real time.

Bypass is fast and acts as an HTTP transactions basis.
Even if the web site is personalized, mostly main page(.html) changes dynamically and the rest of 99% is made up of static resources.
Bypass version of :ref:`origin-httprequest` is provided in order to return the identical response with the intention of origin server.


.. toctree::
   :maxdepth: 2


No-Cache Request Bypass
====================================

If a client sends no-cache request, bypass the request. ::

   GET / HTTP/1.1
   cache-control: no-cache or cache-control:max-age=0
   pragma: no-cache
    
::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BypassNoCacheRequest>OFF</BypassNoCacheRequest>   
    
-  ``<BypassNoCacheRequest>``

   - ``OFF (default)`` Cache module handles the request.
   
   - ``ON`` Bypass the request to the origin server.
   
.. note::

    This configuration is judged by client's action(probably ``ctrl`` + ``F5`` ).
    Therefore, excessive number of bypass might increase origin load.
    

.. _bypass-getpost:    
 
GET/POST Bypass
====================================

Bypass can be set a default reaction for GET/POST requests.
The default reaction of the two requests might be different as GET and POST have different usages. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BypassPostRequest>ON</BypassPostRequest>
   <BypassGetRequest>OFF</BypassGetRequest>   
    
-  ``<BypassPostRequest>``

   - ``ON (default)`` Bypass POST requests to the origin server.
   
   - ``OFF`` STON handles POST requests.
   
-  ``<BypassGetRequest>``

   - ``OFF (default)`` STON handles GET requests.

   - ``ON`` Bypass GET requests to the origin server.

Supports all condition as :ref:`access-control-vhost_acl`.
Exception for bypass is saved at /svc/{virtual host name}/bypass.txt. ::

   # /svc/www.example.com/bypass.txt
   $IP[192.168.2.1-255]
   /index.html   
    
If cache or bypass condition has not been specified, the opposite setting of default condition will be applied.
For example, if ``<BypassGetRequest>`` is set to ``ON``, the exceptional condition is a Caching list.
If this is confusing, you can use second parameter to make the condition more clear. ::

   # /svc/www.winesoft.co.kr/bypass.txt
   
   $HEADER[cookie: *ILLEGAL*], cache               // Always cache
   !HEADER[referer:]                               // Depends on the default setting
   !HEADER[referer] & !HEADER[user-agent], bypass  // Always bypass
   $URL[/source/public.zip]                        // Depends on the default setting

The priority will be as below.

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

   - ``ON (default)`` This setting guarantees to bypass client requests to an identical server. 
     But, it might not be an identical socket. 
     
     The origin server and all connected sockets could lose connections.
     In this case, new socket connections will be requested to the relative server.
     
     .. figure:: img/private_bypass3.jpg
        :align: center
      
        Requests are always bypassed to the same server.
     
     If bypassed origin server is inactive due to errors or excluded from DNS, another server will be used for bypass.
   
   - ``OFF`` This setting does not guarantee which server will be used for client requests.
   
     .. figure:: img/private_bypass1.jpg
        :align: center
      
        Abides by :ref:`origin-balancemode`.
        


Origin Session Affinity
====================================

Each client socket use 1:1 bypass session with the origin server. 

.. figure:: img/private_bypass2.jpg
   :align: center
      
   Client owns origin session.

Origin session can be fixed with `GET/POST Bypass`_ property. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BypassPostRequest Private="OFF">...</BypassPostRequest>
   <BypassGetRequest Private="OFF">...</BypassGetRequest>   
    
-  ``Private``

   - ``OFF (default)`` This setting let client session use a dedicated origin server session.
     Requests are always bypassed to the same server.
     Either client or origin server terminates the session, other party's session will be also closed.
        
   - ``OFF`` This setting does not use a dedicated session.

Just like the origin server keeps client's login information based on the session, 
it is useful when the client request must be handled with the same socket.

.. note::

   If too many requests are bypassed with ``Private``, the origin server could be loaded as much as the number of client.  
   Also, these origin sessions are owned by clients and this could endanger the server from melicious attacks.
   

Timeout
-----------------------

Bypass usually responds a result that is dynamically processed from the origin server.
Therefore, the processing speed is slower than static contents.
Setting a ``Timeout`` only for bypass is recommended to prevent hastily assuming as an error condition. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <BypassConnectTimeout>5</BypassConnectTimeout>
   <BypassReceiveTimeout>300</BypassReceiveTimeout>   

-  ``<BypassConnectTimeout> (default: 5 seconds)``   
   If the bypass connection with origin server is not established for the set amount of time, it is handled as a connection timeout.


-  ``<BypassReceiveTimeout> (default: 5 seconds)``
   If the origin server does not respond for the set amount of tiem during bypass, it is handled as a receive timeout.
   
   

Bypass Header
====================================

Configure whether to apply the bypass setting of :ref:`origin-httprequest`. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <UserAgent Bypass="OFF">...</UserAgent>
   <Host Bypass="ON"/>
   <XFFClientIPOnly Bypass="ON">...</XFFClientIPOnly>   
    
-  ``Bypass`` Property

   - ``ON`` Specifies configured header.
        
   - ``OFF`` Specifies relative header from clients.


.. _bypass-port:

Port Bypass
====================================

Bypasses all packets of a specific TCP port to the origin server.
This setting is exclusive for virtual host. ::

   # vhosts.xml - <Vhosts>
      
   <Vhost Name="www.example">
      <PortBypass>443</PortBypass>
      <PortBypass Dest=”1935”>1935</PortBypass>
   </Vhost>

-  ``<PortBypass>``   
   Bypasses all packets from a designated port to the same port of the origin server.
   ``Dest`` property will configure the origin server port.

For example,if you are bypassing 443 port, the result is similar to establishing a direct SSL connection between the client and origin server. 
The port that is being bypassed can never have redundant setting. 

.. note::
   
   Structurally port bypass is handled in a TCP Layer that is lower than HTTP. 
   The reason of configuring a port bypass under a specific virtual host is that a subject to collect statistics is required.
   

