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

If you set the origin address with an IP, the setting is much simpler than using a domain. 

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

Even if you set the origin address as a domain, error/recovery features work based on IP address.
DNS client(STON) does not identify all IP addresses in the domain. 
However, if the domain consists of unavailable IP addresses, server failure status cannot last
(??이 문장에 대한 구체적인 설명이 필요합니다.)하지만 사용할 수 없는 IP들만으로 Domain을 구성할 경우 장애상태가 지속될 수 없다.

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
   Various IPs and TTLs are used.
   
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

   When the STON downloads contents for the first time, ``Content-Length`` is unknown so the ``Range`` cannot be requested.
   If ``<PartSize>`` is configured for the contents, only configured size of contents will be downloaded before connection is closed.
   
      


Initializing Entire Range
====================================

When the STON for the first time  downloads or checks modification from the origin server, the simple form of ``GET`` request is sent. ::

    GET /file.dat HTTP/1.1
    
However, if the origin server is configured to modulate files for general ``GET`` requests, this could be an issue because original file cannot be cached.

One of the most common examples is that the Apache web server embeds external modules such as mod_h.264_streaming. 
Apache web server always response to ``GET`` requests via mod_h.264_streaming module.
Client(STON, in this case) is being serviced with modulated file by mod_h.264_streaming module.

   .. figure:: img/conf_origin_fullrangeinit1.png
      :align: center
      
      mod_h.264_streaming module always modulate original files.

Range request bypasses the module and downloads original files. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <FullRangeInit>OFF</FullRangeInit>   

-  ``<FullRangeInit>``

   - ``OFF (default)`` Transmit general HTTP request.
   
   - ``ON`` Transmit the Range request that starts from 0. 
     Apache will bypass modules if Range header is specified. ::
      
        GET /file.dat HTTP/1.1
        Range: bytes=0-
    
     When caching file for the first time, Full-Range(starting from 0) will be requested because the Range of contents is unknown.
     You should check whether the origin server responds to the ``Range`` request with normal response(206 OK).

When contents are being modified, **If-Modified-Since** header will be specified as below.
The origin server must properly reply with **304 Not Modified**. ::

   GET /file.dat HTTP/1.1
   Range: bytes=0-
   If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT
    
.. note::

   The list of web server that ``<FullRangeInit>`` is working correctly.
    
   - Microsoft-IIS/7.5
   - nginx/1.4.2
   - lighttpd/1.4.32
   - Apache/2.2.22
    

.. _origin-httprequest:
    
Origin Request Header
====================================

Host Header
---------------------

Configure the Host header of HTTP request that will be sent to the origin server.
If not configured, virtual host name will be specified. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <Host />   

-  ``<Host>``
   Configure the Host header that will be sent to the origin server.
   If the origin server is using some other port than 80, you should also specify the port number. ::
   
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

If STON is placed in between the client and the origin server, the origin server cannot obtain client's IP address.
Therefore, STON specifes X-Forwarded-For header to all HTTP requests that are sent to the origin server. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <XFFClientIPOnly>OFF</XFFClientIPOnly>   

-  ``<XFFClientIPOnly>``
   
   - ``OFF (default)`` Append client's IP to the XFF header that is received from the client(IP: 128.134.9.1).
     If the client did not send XFF header, only the client IP will be specified. ::
      
        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1
   
   - ``ON`` Send the first address of XFF header to the origin server. ::
   
        X-Forwarded-For: 220.61.7.150


Identifying ETag Header
---------------------

The following shows hot to configure whether or not to recognize the ETag that will be responded by the origin server. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <OriginalETag>OFF</OriginalETag>   

-  ``<OriginalETag>``
   
   - ``OFF (default)`` Ignores the ETag header.
   
   - ``ON`` Identifies ETag and append If-None-Match header when updating contents.

   
Redirect Tracking
====================================

If the origin server replies with Redirect responses(301, 302, 303, 307), tracks Location header and requests contents. 

   .. figure:: img/conf_redirectiontrace.png
      :align: center
      
      Clients does not know whether they are redirected or not.

::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <RedirectionTrace>OFF</RedirectionTrace>   

-  ``<RedirectionTrace>``

   - ``OFF (default)`` Saved as 3xx response.
   
   - ``ON`` Download contents from the address in Location header.
     If the format of redirection response is incorrect or the Location header is missing, tracking will not work.
     In order to prevent infinite rediretion, the STON only tracks once.

