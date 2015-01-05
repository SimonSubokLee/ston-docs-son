.. _handling_http_requests:

Chapter 6. Handling HTTP Requests
******************

This chapter explains methods to handle HTTP client sessions and requests.
The contents in this chapter are not critical for the service.
Some of the contents might be difficult to understand if you don't have basic understanding of HTTP.
In this case, you can simply use default setting as it will not affect to the quality of service at all.


.. toctree::
   :maxdepth: 2


Session Management
====================================

An HTTP session is created when an HTTP client is connected to the STON server.
The client is service through the HTTP session with various contents that are saved in the server.
**HTTP transaction** stands for the procedure from request to response.
The HTTP session handles multiple HTTP transactions in order. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ConnectionHeader>keep-alive</ConnectionHeader>
   <ClientKeepAliveSec>10</ClientKeepAliveSec>
   <KeepAliveHeader Max="0">ON</KeepAliveHeader>   
    
-  ``<ConnectionHeader> (default: keep-alive)``    
   Configures Connection header(``keep-alive`` or ``close``) of HTTP response that will be sent to clients.
    

-  ``<ClientKeepAliveSec> (default: 10 seconds)``
   Terminates session when there is no transaction with the client session for the set amount of time.
   If you set longer time for this option, there will be more alive sessions that are not transacting with clients.
   Having too many sessions will increase load of the system.

-  ``<KeepAliveHeader>``

    - ``ON (default)`` Specifies Keep-Alive header in the HTTP response.
      ``Max (default: 0)`` If this option is set to greater than 0, ``Max`` value will be used for Keep-Alive header.
      Every HTTP transaction will decrease the value by 1.
   
   - ``OFF`` Omitts Keep-Alive header in the HTTP response.


HTTP Session Maintenance Policies
---------------------

STON preferably abides by policies of Apache.
Especially session maintenance policy varies by HTTP header values.
The followings are the items that affect HTTP session maintenance policy.

- The connection header that is specified in the client HTTP request ("ep-Alive" or "Close").
- Virtual host ``<Connection>`` setting
- Virtual host session Keep-Alive time setting
- Virtual host ``<Keep-Alive>`` setting


1. When "Connection: Close" is specified in the client HTTP request ::

      GET / HTTP/1.1
      ...(skip)...
      Connection: Close
    
   For the HTTP request like this, "Connection: Close" will be returned regardless of the virtual host configuration. 
   Keep-Alive header will not be specified. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Close

   When this HTTP transaction is completed, disconnect the HTTP connection.
   

2. When ``<ConnectionHeader>`` is set to ``Close`` ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Close</ConnectionHeader>      
    
   "Connection: Close" will be returned regardless of the HTTP request from clients. 
   Keep-Alive header will not be specified. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Close
      

3. When ``<KeepAliveHeader>`` is set to ``OFF`` ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader>OFF</KeepAliveHeader>
    
   Kepp-Alive header will not be specified. HTTP session can be reused. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive


4. When ``<KeepAliveHeader>`` is set to ``ON`` ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader>ON</KeepAliveHeader>      
    
   Keep-Alive header will be specified.
   Keep-Alive value of the session will be used for timeout. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10

   .. note::

      < ``<Keep-Alive>`` and ``<ClientKeepAliveSec>`` >
    
      ``<Keep-Alive>`` setting refers to ``<ClientKeepAliveSec>`` that has more fundamental purpose.
      One of the most important issues to keep high performance and more available resources is determining when to terminate idle sessions(sessions that does not generate HTTP transactions).
      HTTP header setting can be changed dynamically or omitted, but terminating idle sessions is more complicated issue. 
      Thereore, ``<ClientKeepAliveSec>`` is separated from ``<KeepAliveHeader>``.


5. When ``<KeepAliveHeader>`` includes ``Max`` property ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader Max="50">ON</KeepAliveHeader>      
    
   Max value will be specified in the Keep-Alive header. 
   This session can be used for the number of times set by ``Max`` property, and every HTTP transaction will decrease the value by 1. ::
    
      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=50


6. When the max value of Keep-Alive is consumed ::

   If max value is set from the above configuration, the value will be gradually diminished by 1 as below. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=1
    
   The above response means one last HTTP transaction is available for current session. 
   If there is another HTTP request for this session, "Connection: Close" will be returned as below. ::
    
      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Close    



Client Cache-Control
====================================

This section explains the configuration regarding to the client cache-control.

Age Header
---------------------

Age header stands for the elapsed time(in second) from cached moment, and calculated by `RFC2616 - 13.2.3 Age Calculations <http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.2.3>`_. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <AgeHeader>OFF</AgeHeader>   
    
-  ``<AgeHeader>``
    
   -  ``OFF (default)`` Omits Age header.
   
   -  ``OFF`` Specifies Age header.


Expires Header
---------------------

The following will refresh Expires header. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <RefreshExpiresHeader Base="Access">OFF</RefreshExpiresHeader>   
    
-  ``<RefreshExpiresHeader>``
    
   -  ``OFF (default)`` Specifies Expires header that is returned from the origin server to the client.
      If Expires header is omitted in the origin server, it will also be omitted in the client response.
   
   -  ``ON``  Expires condition will be reflected to the Expires header.
      Any contents that do not meet the condition will be applied ``OFF (default)`` setting.
   
The Expires condition is identical to `mod_expires <http://httpd.apache.org/docs/2.2/mod/mod_expires.html>`_ of Apache. 
You can also configure Expires header and Cache-Control value of contents with special conditions(URL or MIME Type). 
The max-age value of the Cache-Control is equals to the value of Expires time subtracted by requested time. 

Expires condition is saved at /svc/{virtual host name}/expires.txt. ::

   # /svc/www.exmaple.com/expires.txt
   # The identifier is a comma(,), and {condition},{time},{criteria} format is used.

   $URL[/test.jpg], 86400
   /test.jpg, 86400
   *, 86400, access
   /test/1.gif, 60 sec
   /test/*.dat, 30 min, modification
   $MIME[application/shockwave], 1 years
   $MIME[application/octet-stream], 7 weeks, modification
   $MIME[image/gif], 3600, modification

-  **Conditions**

   URL and MIME Type can be used. 
   $URL[...] is used for URL, and $MIME[...] is used for MIME Type. 
   Patterned expression is also available and if $ omitted, it is recognized as a URL.

-  **Time**

   Set the Expires expiration time. 
   General units of time are supported, and if the unit is not specified, second will be used.

-  **Criterion**

   Configures reference time of expiration time of Expires. 
   If the reference time is not specified, Access will be used for the reference time. 
   Access refers current time. 
   The following is an example of configuring Expires header value. 
   If MIME Type accesses image/gif file, Expire header will be set as 1 day and 12 hours. ::
    
      $MIME[image/gif], 1 day 12 hours, access
      
   Modification is based on the Last-Modified sent from the source server. 
   The following example shows how to set the Expires to 30 minutes for all jpg files from Last-Modified. ::
    
      *.jpg, 30min, modification
        
   In case of Modification, if calculated Expires value is older than current time, adjust it as current time.
   If origin server does not provide Last-Modified header, Expires header will not be sent.


ETag Header
---------------------

This section explains how to specify an ETag header in the HTTP resonse sent to the client. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ETagHeader>ON</ETagHeader>   
    
-  ``<ETagHeader>``
    
   -  ``ON (default)`` Specifies ETag header.
   
   -  ``OFF``  Omitts ETag header.
   
   


Response Headers
====================================

HTTP Request / Response Header Modification
---------------------

This section explains how to modify the client HTTP request and response based on specific condition. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>   
    
-  ``<ModifyHeader>``
    
   -  ``OFF (default)`` Does not modify.
   
   -  ``ON`` Modifies header regard to the header modification condition.
   
You should be able to identify the moment when the header has to be modified.

-  **HTTP Request Header Modification Point**

   You should modify the header when the client HTTP request is initially identified. 
   If header has been modified, it will be processed in the cache module as it is.
   However, Host header and URI cannot be modulated.

-  **HTTP Response Header Modification Point**

   You should modify the header right before responding to the client. 
   However, the Content-Length cannot be modified.   

      
Modify condition of header is saved at /svc/{virtual host name}/headers.txt. 
The header allows multiple configurations as long as it fits to the condition. 
All modification will be applied at the same time. 

If you wish to modify the first condition only, set the ``FirstOnly`` property to ``ON``.
When different conditions try to modify an identical header, it'll be either Last-Win or specifically appended. ::

   # /svc/www.example.com/headers.txt
   # The comma(,) is an identifier.
   
   # Request Modification
   # {Match}, {$REQ}, {Action(set|unset|append)} format is used.
   $IP[192.168.1.1], $REQ[SOAPAction], unset
   $IP[192.168.2.1-255], $REQ[accept-encoding: gzip], set
   $IP[192.168.3.0/24], $REQ[cache-control: no-cache], append
   $IP[192.168.4.0/255.255.255.0], $REQ[x-custom-header], unset
   $IP[AP], $REQ[X-Forwarded-For], unset
   $HEADER[user-agent: *IE6*], $REQ[accept-encoding], unset
   $HEADER[via], $REQ[via], unset
   $URL[/source/*.zip], $REQ[accept-encoding: deflate], set
   
   # Response Modification
   # {Match}, {$RES}, {Action(set|unset|append)}, {condition} format is used.
   # {condition} modifies the header regarding to a specific response code, but it is not a mandatory.
   $IP[192.168.1.1], $RES[via: STON for CDN], set
   $IP[192.168.2.1-255], $RES[X-Cache], unset, 200
   $IP[192.168.3.0/24], $RES[cache-control: no-cache, private], append, 3xx
   $IP[192.168.4.0/255.255.255.0], $RES[x-custom-header], unset
   $HEADER[user-agent: *IE6*], $RES[vary], unset
   $HEADER[x-custom-header], $RES[cache-control: no-cache, private], append, 5xx
   $URL[/source/*], $RES[cache-control: no-cache], set, 404
   /secure/*.dat, $RES[x-custom], unset, 200
    
IP, GeoIP, Header and URL form can be used for {Match} configuration.

-  **IP**
   $IP[...] format is used, and supports IP, IP Range, Bitmask and Subnet formats.

-  **GeoIP**
   $IP[...] format is used and :ref:`access-control-geoip` must be predefined.
   Country codes of `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ and `ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ are supported.
     
-  **Header**
   $HEADER[Key : Value] format is used. 
   Value supports specific expressions and patterns. 
   Value가 생략된 경우에는 Key에 해당하는 헤더의 존재유무를 (어떠한 조건인지???)조건으로 판단한다.
    
-  **URL**
   $URL[...] format is used and can be omitted. Specific expressions and patterns will be recognized.
    
{$REQ} and {$RES} configure how to modify the header.
Generally ``set`` and ``append`` use {Key: Value} for configuration, and if the Value is omitted, empty value("") will be inserted. 
For ``unset``, only {Key} value is required.

``set`` , ``unset`` , ``append`` can be used for {Action}.

-  ``set``  Add Key and Value that are defined in the request/response header to the header. 
   If an identical Key exists, overwrites the previous value.    

-  ``unset`` Discard the header related to the Key that is defined in the request/response header.

-  ``append``  This is similar to ``set``, while this setting uses comma(,) to combine previous Value with configured Value when related Key is found.

{Condition} identifies response code families such as 2xx, 3xx, 4xx and 5xx instead of specific response codes such as 200 and 304.
If {Condition} is not matching, modification will not be reflected even if {Match} is matching.
If {Condition} is omitted, response code will not be examined.


Original Header
---------------------

In order to keep the decent performance, only standard header will be selectively identified from headers that origin server transmit. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <OriginalHeader>OFF</OriginalHeader>   
    
-  ``<OriginalHeader>``

   -  ``OFF (default)`` Ignores if the header is not a standard. 
   
   -  ``OFF``  Saves non-standard header and transfer it to the client.
      However, this option will consume more memory and storage.


Via Header
---------------------

This section explains how to configure whether to specify Via header or not in the HTTP response that will be sent to the client. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ViaHeader>ON</ViaHeader>   
    
-  ``<ViaHeader>``
    
   - ``ON (default)`` Specifies Via header as below.
     ::
      
        Via: STON/2.0.0
   
   - ``OFF``  Omits Via header.
   
   
Server Header
---------------------
 
This section explains how to configure whether to specify Server header or not in the HTTP response that will be sent to the client. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ServerHeader>ON</ServerHeader>   
    
-  ``<ServerHeader>``
    
   -  ``ON (default)`` Specifies Server header of the origin server. ::
   
   -  ``OFF``  Omits Server header.



URL Preprocessing
====================================

`Regular expression <http://en.wikipedia.org/wiki/Regular_expression>`_ is used to modify requested URL. 
If URL preprocessing is defined, all client requests(HTTP or File I/O) should pass the URL Rewriter.

.. figure:: img/urlrewrite1.png
   :align: center
      
   After passing the URL Rewriter, virtual host can be accessed.
   
If an approaching Host name has been modified by URL Rewriter, it is considered that the Host header of client HTTP request has been modified.
URL preprocessing is configured at the virtual host setting(vhosts.xml).
Most configurations are subordinated to the virtual host, but URL preprocessing can modify Host name that a client requested so it should be configured in the same block as virtual host. ::

   # vhosts.xml
   
   <Vhosts>
      <Vhost ...> ... </Vhost>
      <Vhost ...> ... </Vhost>
      <URLRewrite ...> ... </URLRewrite>
      <URLRewrite ...> ... </URLRewrite>
   </Vhosts>
    
Multiple configurations are allowed, and regular expression will be checked in order. ::

   # vhosts.xml - <Vhosts>
   
   <URLRewrite AccessLog="Replace">
       <Pattern>www.exmaple.com/([^/]+)/(.*)</Pattern>
       <Replace>#1.exmaple.com/#2</Replace>
   </URLRewrite>
    
-  ``<URLRewrite>``

   Configures URL preprocessing.
   ``AccessLog (default: Replace)`` attribute configures URLs that will be recorded in the Access log. 
   ``Replace`` records preprocessed URL(/logo.jpg), whereas ``Pattern`` records unprocessed URL(/baseball/logo.jpg) to the Access log.
   
   -  ``<Pattern>`` configures patterns to be matched. 
      A single pattern is expressed with parenthesis(eg. ( )).
   
   -  ``<Replace>`` configures convert format. 
      Identical pattern can use expressions like #1, #2. #0 stands for the entire URL that is requested. 
      Maximum 9 patternes(#9) can be configured.
      
:ref:`monitoring_stats` provides throughput, and :ref:`api-graph-urlrewrite` can be used as well. 
URL preprocessing simplifies expressions with other functions such as :ref:`media-trimming` and :ref:`media-hls`. ::

   # vhosts.xml - <Vhosts>

   <URLRewrite>
       <Pattern>example.com/([^/]+)/(.*)</Pattern>
       <Replace>example.com/#1.php?id=#2</Replace>
   </URLRewrite>
   // Pattern : example.com/releasenotes/1.3.4
   // Replace : example.com/releasenotes.php?id=1.3.4

   <URLRewrite>
       <Pattern>example.com/download/(.*)</Pattern>
       <Replace>download.example.com/#1</Replace>
   </URLRewrite>
   // Pattern : example.com/download/1.3.4
   // Replace : download.example.com/1.3.4

   <URLRewrite>
       <Pattern>example.com/img/(.*\.(jpg|png).*)</Pattern>
       <Replace>example.com/#1/STON/composite/watermark1</Replace>
   </URLRewrite>
   // Pattern : example.com/img/image.jpg?date=20140326
   // Replace : example.com/image.jpg?date=20140326/STON/composite/watermark1

   <URLRewrite>
       <Pattern>example.com/preview/(.*)\.(mp3|mp4|m4a)$</Pattern>
       <Replace><![CDATA[example.com/#1.#2?&end=30&boost=10&bandwidth=2000&ratio=100]]></Replace>
   </URLRewrite>
   // Pattern : example.com/preview/audio.m4a
   // Replace : example.com/audio.m4a?end=30&boost=10&bandwidth=2000&ratio=100

   <URLRewrite>
       <Pattern>example.com/(.*)\.mp4\.m3u8$</Pattern>
       <Replace>example.com/#1.mp4/mp4hls/index.m3u8</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4.m3u8
   // Replace : example.com/video.mp4/mp4hls/index.m3u8

   <URLRewrite>
       <Pattern>example.com/(.*)_(.*)_(.*)</Pattern>
       <Replace>example.com/#0/#1/#2/#3</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4_10_20
   // Replace : example.com/example.com/video.mp4_10_20/video.mp4/10/20
    
If 5 XML special characters( " & ' < > ) are used for patterned expression, you must keep them in <![CDATA[ ... ]]>.
When configuring with :ref:`wm`, all patterns are processed as CDATA.
