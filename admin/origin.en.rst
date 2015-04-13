.. _origin:

Chapter 7. Origin Server
******************

This chapter will explain the relationship between STON and the origin server.
The origin server generally stands for the web server that abides by HTTP standard.
In order to protect the origin server, administrators should have a thorough understanding of the contents in this chapter.
Doing so will also enable you to establish a durable and flexible service that can resist errors in the origin server.

There are many plans for dealing with the various errors that may occur, but the bottom line is that the origin server must be protected. 
Having a proper protection policy for the origin server will make your server inspection procedure worry free. 

.. toctree::
   :maxdepth: 2



.. _origin_exclusion_and_recovery:

Error Detection and Recovery
====================================

If an origin server failure occurs during caching, the server will be automatically excluded.
When the server is recovered, it'll be utilized for the service. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
    
   <ConnectTimeout>3</ConnectTimeout>
   <ReceiveTimeout>10</ReceiveTimeout>
   <Exclusion>3</Exclusion>
   <Recovery Cycle="10" Uri="/" ResCode="0" Log="ON">5</Recovery>   

-  ``<ConnectTimeout> (default: 3 seconds)``
   
   If the origin server is not connected within the set amount of seconds, it is considered as a connection failure.

-  ``<ReceiveTimeout> (default: 10 seconds)``
   
   If the origin server does not reply HTTP response within the set amount of seconds for a normal HTTP request, it is considered as transaction failure.

-  ``<Exclusion> (default: 3 times)``
   
   If the set amount of consecutive failures( ``<ConnectTimeout>`` or ``<ReceiveTimeout>`` ) occur in the origin server, the related server will be excluded from the available server list. 
   This value will be reset to 0 if a successful communication occurs before exclusion.

-  ``<Recovery> (default: 5 times)``
   
   Request with ``Uri`` in every ``Cycle``, and if the origin server replies ``ResCode`` for the set amount of consecutive times, then the connection to the origin server is recovered.
   Setting this value to 0 will not recover the server.   
   
   -  ``Cycle (default: 10 seconds)`` Tries a new request every configured amount seconds.
   
   -  ``Uri (default: /)`` Sets Uri to send a request.
   
   -  ``ResCode (default: 0)`` The response code that will be identified as a normal response.
      Setting this value to 0 will regard any reply as a success.
      For example, setting this value to 200 will process only 200 response as a success.
      Multiple valid response codes can be configured by using a comma(,) to separate each code.
      For example, each one of the numbers listed in the configuration "200, 206, 404" value will be regarded as a success.

   -  ``Log (default: ON)`` Records the HTTP transaction that was used for recovery to :ref:`admin-log-origin`.
      
      

.. _origin-health-checker:

Health-Checker
====================================

`Error Detection and Recovery`_ responds to failures during the caching process.
``<Recovery>`` terminates an HTTP transaction as soon as a response code is received.
However, Health-Checker checks for a successful HTTP transaction. ::

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
   Uri is used as a value, and CDATA is used for XML exception characters.
   
   -  ``ResCode (default: 0)`` Correct response codes. Multiple configurations are allowed with commas(,) used to separate each code.
   
   -  ``Timeout (default: 10 seconds)`` A valid time, from the socket connection untill the HTTP transaction is completed.

   -  ``Cycle (default: 10 seconds)`` An execution period.
   
   -  ``Exclusion (default: 3 times)`` The specified number of times to fail before excluding the related server.
   
   -  ``Recovery (default: 5 times)`` The number of consecutive successes before deploying the server.
   
   -  ``Log (default: ON)`` Records an HTTP transaction to :ref:`admin-log-origin` .

Health-Checker can have multiple configurations, and can be executed independently regardless of client requests.
It does not share information with the `Error Detection and Recovery`_ or other Health-Checkers to decide exclusion and deployment.


.. _origin-use-policy:

Origin Address Use Policy
====================================

The usage of the Origin address(IP) is determined according to the following elements.

-  :ref:`env-vhost-activeorigin` Address format(IP or Domain) and standby address
-  `Error Detection and Recovery`_
-  `Health-Checker`_

The origin address is frequently excluded/recovered when running a service.
STON uses an IP table-based origin address, and information can be accessed by an `origin-status`_ API.

If you set the origin address with an IP, the setting is much simpler than using a domain. 

-  Nothing will modify the IP list unless you change the configuration.
-  The IP address will not be expired by TTL.
-  Exclusion/recovery works based on the IP address.

If the origin address is set as a domain, you have to resolve it to acquire an IP.
( recorded in :ref:`admin-log-dns` .)
An IP list can be changed dynamically, and all IPs are only valid for the valid TTL.

-  The domain is periodically resolved (1~10 seconds).
-  Organize an IP table based upon the resolved result.
-  All IPs are valid as long as the TTL is valid. IPs will not be used if the TTL is expired.
-  If an identical IP is resolved, the TTL will be renewed.
-  The IP table should not be cleared even if the TTL is expired. The most recent IPs will not be discarded.

Even if you set the origin address as a domain, the error/recovery features work based on the IP address.
The DNS client (which is STON) can not identify changes of IP addresses in the domain. 
However, if the domain consists of unavailable IP addresses, server failure cannot be properly processed.

Domain address error/recovery policies are listed below:

-  If all known IP addresses for a domain are inactivated, the related domain address will also be inactive.
-  Even if a new IP is acquired from the resolving process, if the domain is inactive, the IP will also be inactivated.
-  If the TTLs of all the IPs are expired, the inactive domain will not be reactivated.
-  At least one of the IP addresses of the inactive domain should be recovered in order to reactivate the domain.

It might be quite difficult to understand, but the `origin-status`_ API will help you to get to know more about the service operation status.



.. _origin-status:

Origin Status Monitoring
====================================

An API is used to monitor the origin status of the virtual host. ::

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
    
-  ``VirtualHost`` The virtual host name

-  ``Address`` :ref:`env-vhost-activeorigin` .
   ``Active`` will be returned if the configured address is in use, and ``Inactive`` will be returned when the address is not in use due to an error.

-  ``Address2`` :ref:`env-vhost-standbyorigin` .
   ``Active`` will be returned if the configured address is in use, otherwise ``Inactive`` will be returned.

-  ``ActiveIP`` the IP list and TTL that are in use. 
   If the origin server is set by IP address, an identical IP ``Address`` with a TTL value of 0 will be returned.
   If it is set by domain, the return value depends on the Resolving result.
   Various IPs and TTLs are used.
   
-  ``InactiveIP`` The IP list and TTL that are not in use.
   Even though the IP is not in use, it could be in a recovery status or be managed by HealthChecker.
   If the address is not recovered wihtin the TTL, it'll be removed.
   

    
.. _origin-status-reset:

Origin Status Reset
====================================

An API is used to reset the origin server exclusion/recovery of the virtual host. 
Also, the current session will not be reused, as a new connection will be created instead. ::

   http://127.0.0.1:10040/command/resetorigin       // All virtual hosts
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com   



.. _origin-busysessioncount:

Overload Judgement
====================================

Content that is requested for the first time, must always be retrieved from the origin server.
On the other hand, if the content is already cached, the request can be more flexibly responded to.
If the server is already overloaded, renewal is postponed to maintain low origin load. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <BusySessionCount>100</BusySessionCount>   

-  ``<BusySessionCount> (default: 100)``
   If the number of HTTP transactions in progress with the origin server exceeds a certain number, it will be judged as a overload.
   In order to block any contents renewal requests to the origin server during the overload status, extend the TTL for ``<OriginBusy>`` from :ref:`caching-policy-ttl`.
   You can set a very large value for this option to forward all requests unconditionally to the origin server.
   

.. _origin-balancemode:

Origin Selection
====================================

When the origin server consists of multiple addresses(two or more addresses), configure the origin sever selection policy. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>RoundRobin</BalanceMode>   

-  ``<BalanceMode> (default: RoundRobin)``

   -  ``RoundRobin (default)`` 
      Round-Robin is applied so that all origin servers have equal access to requests.
      Connected Idle session is only used when a request is need to the related server.
   
   -  ``Session``      
      If there are any reusable sessions, make use of them. 
      If new session are required, use Round-Robin to allocate sessions to the server.
      
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
However, the origin server can unilaterally close the connection for a request from a recycled session.
Therefore, reestablishing the connection might cause a delay in user reactivity, especially for sessions that have not been reused for a long time.
The following configuration will close the connection of unrecycled sessions for n seconds. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <ReuseTimeout>60</ReuseTimeout>   

-  ``<ReuseTimeout> (default: 60 seconds)`` 
   Closes an origin session that has not been used for the set amount time.
   Setting this value to 0 will not reuse the origin server session.
   
   
.. _origin_partsize:

Range Request
====================================

Configure the size of content to download.
In cases of content that are usually consumed from the head of file (like video clips), restricting the download size can reduce unnecessary origin traffic. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <PartSize>0</PartSize>   

-  ``<PartSize> (default: 0 MB)``
   If this value is greater than 0, the configured size will be downloaded by using a range request from the client's location.   


``<PartSize>`` can also help save disk space.
STON generates a same-sized file as the original file in the disk.
If ``<PartSize>`` is not 0, downloaded files will be partitioned and saved.

For example, if a client watches the first minute(10MB) of a one-hour(600MB) video clip, only 10MB of disk space is used.
This option will save disk space, while partitioning a file will increase the disk load a bit.

.. note::

   When STON downloads content for the first time, ``Content-Length`` is unknown so the ``Range`` cannot be requested.
   If ``<PartSize>`` is configured for the contents, only the configured size of contents will be downloaded before the connection is closed.
   
      


Initializing Entire Range
====================================

When STON downloads or checks modifications from the origin server for the first time, the simple form of ``GET`` request is sent. ::

    GET /file.dat HTTP/1.1
    
However, origin servers configured to modulate files for general ``GET`` requests may have issues because the original file cannot be cached.

One of the most common examples is that the Apache web server embeds external modules such as mod_h.264_streaming. 
The Apache web server always responds to ``GET`` requests via a mod_h.264_streaming module.
Below, a client(STON, in this case) is being serviced with a modulated file by a mod_h.264_streaming module.

   .. figure:: img/conf_origin_fullrangeinit1.png
      :align: center
      
      The mod_h.264_streaming module always modulates original files.

A range request bypasses the module and downloads original files. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <FullRangeInit>OFF</FullRangeInit>   

-  ``<FullRangeInit>``

   - ``OFF (default)`` Transmits a general HTTP request.
   
   - ``ON`` Transmits Range requests that starts from 0. 
     Apache will bypass modules if a Range header is specified. ::
      
        GET /file.dat HTTP/1.1
        Range: bytes=0-
    
     When caching a file for the first time, Full-Range(starting from 0) will be requested because the content's Range is unknown.
     You should check whether the origin server responds to the ``Range`` request with a normal response(206 OK).

When content is being modified, an **If-Modified-Since** header will be specified, as shown below.
The origin server must properly reply with **304 Not Modified**. ::

   GET /file.dat HTTP/1.1
   Range: bytes=0-
   If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT
    
.. note::

   ``<FullRangeInit>`` works best with the following web servers.
    
   - Microsoft-IIS/7.5
   - nginx/1.4.2
   - lighttpd/1.4.32
   - Apache/2.2.22
    

.. _origin-httprequest:
    
Origin Request Header
====================================

Host Header
---------------------

Configure the Host header of an HTTP request that will be sent to the origin server.
If not configured, the virtual host name will be specified. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <Host />   

-  ``<Host>``
   Configures the Host header that will be sent to the origin server.
   If the origin server is using a port other than 80, you should also specify the port number. ::
   
      # server.xml - <Server><VHostDefault><OriginOptions>
      # vhosts.xml - <Vhosts><Vhost><OriginOptions>
      
      <Host>www.example2.com:8080</Host>


If you want to transfer the Host header from a client, use * for this setting.


User-Agent Header
---------------------

Configure the User-Agent header of HTTP request that will e sent to the origin server. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent>STON</UserAgent>   

-  ``<UserAgent> (default: STON)``
   Configure the User-Agent header that will be sent to the origin server.


XFF(X-Forwarded-For) Header
---------------------

If STON is placed in between the client and the origin server, the origin server cannot obtain the client's IP address.
Therefore, STON specifies an X-Forwarded-For header to all HTTP requests that are sent to the origin server. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <XFFClientIPOnly>OFF</XFFClientIPOnly>   

-  ``<XFFClientIPOnly>``
   
   - ``OFF (default)`` Appends the client's IP to the XFF header that is received from the client(IP: 128.134.9.1).
     If the client did not send an XFF header, only the client IP will be specified. ::
      
        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1
   
   - ``ON`` Send the first address of the XFF header to the origin server. ::
   
        X-Forwarded-For: 220.61.7.150


Identifying ETag Header
---------------------

The following shows how to configure whether or not to recognize the ETag that the origin server will respond to. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <OriginalETag>OFF</OriginalETag>   

-  ``<OriginalETag>``
   
   - ``OFF (default)`` Ignores the ETag header.
   
   - ``ON`` Identifies the ETag and appends an If-None-Match header when updating contents.

   
Redirect Tracking
====================================

If the origin server replies with Redirect responses(301, 302, 303, 307), the Location header is tracked to request content. 

   .. figure:: img/conf_redirectiontrace.png
      :align: center
      
      Clients do not know whether they are redirected or not.

::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <RedirectionTrace>OFF</RedirectionTrace>   

-  ``<RedirectionTrace>``

   - ``OFF (default)`` Saves as a 3xx response.
   
   - ``ON`` Downloads content from the address in the Location header.
     If the format of the redirection response is incorrect or the Location header is missing, tracking will not work.
     In order to prevent infinite redirection, the STON only tracks once.

