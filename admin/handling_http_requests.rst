.. _handling_http_requests:

Chapter 5. Handling HTTP Requests
******************

This chapter explains methods to handle HTTP client sessions and requests.
The contents in this chapter are not critical for the service.
Some content might be difficult to follow, if you don't have a basic understanding of HTTP.
In this case, you can simply use the default setting as it will not affect the quality of service at all.


.. toctree::
   :maxdepth: 2


Session Management
====================================

An HTTP session is created when an HTTP client is connected to the STON server.
The client is serviced through the HTTP session with a variety of content that is saved on the server.
**HTTP transaction** stands for the procedure from request to response.
The HTTP session handles multiple HTTP transactions in order. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ConnectionHeader>keep-alive</ConnectionHeader>
   <ClientKeepAliveSec>10</ClientKeepAliveSec>
   <KeepAliveHeader Max="0">ON</KeepAliveHeader>   
    
-  ``<ConnectionHeader> (default: keep-alive)``    
   Configures the Connection header(``keep-alive`` or ``close``) of the HTTP response that will be sent to clients.
    

-  ``<ClientKeepAliveSec> (default: 10 seconds)``
   Terminates a session when there is no transaction with the client session for the set amount of time.
   If you set a longer time for this option, there will be more alive sessions that are not transacting with clients.
   Having too many sessions will increase the load on the system.

-  ``<KeepAliveHeader>``

    - ``ON (default)`` Specifies the Keep-Alive header in the HTTP response
      ``Max (default: 0)`` If this option is set greater than 0, the ``Max`` value will be used for the Keep-Alive header.
      Every HTTP transaction will decrease the value by one.
   
   - ``OFF`` Omits the Keep-Alive header in the HTTP response.


HTTP Session Maintenance Policies
---------------------

STON preferably abides by the policies of Apache.
Especially, session maintenance policy varies depending upon HTTP header values.
The followings are items that affect the HTTP session maintenance policy.

- The connection header that is specified in the client HTTP request ("ep-Alive" or "Close")
- Virtual host ``<Connection>`` setting
- Virtual host session Keep-Alive time setting
- Virtual host ``<Keep-Alive>`` setting


1. When "Connection: Close" is specified in the client HTTP request ::

      GET / HTTP/1.1
      ...(skip)...
      Connection: Close
    
   For an HTTP request like this, "Connection: Close" will be returned regardless of the virtual host configuration. 
   The Keep-Alive header will not be specified. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Close

   When this HTTP transaction is completed, disconnect the HTTP connection.
   

2. When the ``<ConnectionHeader>`` is set to ``Close`` ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Close</ConnectionHeader>      
    
   "Connection: Close" will be returned regardless of clients' HTTP requests. 
   The Keep-Alive header will not be specified. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Close
      

3. When the ``<KeepAliveHeader>`` is set to ``OFF`` ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader>OFF</KeepAliveHeader>
    
   The Keep-Alive header will not be specified. The HTTP session can be reused. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive


4. When the ``<KeepAliveHeader>`` is set to ``ON`` ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader>ON</KeepAliveHeader>      
    
   The Keep-Alive header will be specified.
   The Keep-Alive value of the session will be used for timeout. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10

   .. note::

      < ``<Keep-Alive>`` and ``<ClientKeepAliveSec>`` >
    
      The ``<Keep-Alive>`` setting refers to ``<ClientKeepAliveSec>`` that has a more fundamental purpose.
      One of the most important issues with maintaining high performance and having more available resources is determining when to terminate idle sessions(sessions that do not generate HTTP transactions).
      The HTTP header setting can be changed dynamically or omitted, but terminating idle sessions is a more complicated issue. 
      Therefore, ``<ClientKeepAliveSec>`` is separated from ``<KeepAliveHeader>``.


5. When the ``<KeepAliveHeader>`` includes ``Max`` property ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader Max="50">ON</KeepAliveHeader>      
    
   The max value will be specified in the Keep-Alive header. 
   This session can be used for the number of times set by ``Max`` property, and every HTTP transaction will decrease the value by one. ::
    
      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=50


6. When the max value of Keep-Alive is consumed ::

   If the max value is set from the above configuration, the value will be gradually diminished by one, as shown below. ::

      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=1
    
   The response above means one last HTTP transaction is available for the current session. 
   If there is another HTTP request for this session, "Connection: Close" will be returned, as shown below. ::
    
      HTTP/1.1 200 OK
      ...(skip)...
      Connection: Close    



Client Cache-Control
====================================

This section explains the configuration regarding client cache-control.

Age Header
---------------------

Age header stands for the elapsed time (in seconds) from a cached moment and is calculated by `RFC2616 - 13.2.3 Age Calculations <http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.2.3>`_. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <AgeHeader>OFF</AgeHeader>   
    
-  ``<AgeHeader>``
    
   -  ``OFF (default)`` Omits Age header.
   
   -  ``OFF`` Specifies Age header.


Expires Header
---------------------

The following will refresh the Expires header. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <RefreshExpiresHeader Base="Access">OFF</RefreshExpiresHeader>   
    
-  ``<RefreshExpiresHeader>``
    
   -  ``OFF (default)`` Specifies the Expires header that the origin server returns to the client.
      If the Expires header is omitted in the origin server, it will also be omitted in the client response.
   
   -  ``ON``  The Expires condition will be reflected in the Expires header.
      The ``OFF (default)`` setting will be applied for any content that does not meet the condition.
   
The Expires condition is identical to the `mod_expires <http://httpd.apache.org/docs/2.2/mod/mod_expires.html>`_ of Apache. 
You can also configure the Expires header and Cache-Control value of content with special conditions(URL or MIME Type). 
The max-age value of the Cache-Control is equals to the value of the Expires time subtracted from the requested time. 

The Expires condition is saved at /svc/{virtual host name}/expires.txt. ::

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
   Patterned expression is also available and if $ is omitted, it is recognized as a URL.

-  **Time**

   Set the Expires expiration time. 
   General units of time are supported, and if the unit is not specified, seconds will be used.

-  **Criterion**

   Configures the reference time of the expiration time of Expires. 
   If the reference time is not specified, Access will be used for the reference time. 
   Access refers to the current time. 
   The following configuration is an example for an Expires header value. 
   If MIME Type accesses an image/gif file, the Expires header will be set as 1 day and 12 hours. ::
    
      $MIME[image/gif], 1 day 12 hours, access
      
   Modification is based on the Last-Modified sent from the source server. 
   The following example shows how to set the Expires to 30 minutes for all jpg files from Last-Modified. ::
    
      *.jpg, 30min, modification
        
   In the case of Modification, if a calculated Expires value is older than the current time, adjust it to the current time.
   If the origin server does not provide a Last-Modified header, an Expires header will not be sent.


ETag Header
---------------------

This section explains how to specify an ETag header in the HTTP response sent to the client. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ETagHeader>ON</ETagHeader>   
    
-  ``<ETagHeader>``
    
   -  ``ON (default)`` Specifies ETag header.
   
   -  ``OFF``  Omits ETag header.
   
   


Response Headers
====================================

HTTP Request / Response Header Modification
---------------------

This section explains how to modify a client's HTTP request and response based on specific conditions. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>   
    
-  ``<ModifyHeader>``
    
   -  ``OFF (default)`` Does not modify.
   
   -  ``ON`` Modifies the header with regard to the header modification condition.
   
You should be able to identify the moment when the header has to be modified.

-  **HTTP Request Header Modification Point**

   You should modify the header when the client HTTP request is initially identified. 
   If the header has been modified, it will be processed in the cache module as it is.
   However, the Host header and URI cannot be modulated.

-  **HTTP Response Header Modification Point**

   You should modify the header right before responding to the client. 
   However, the Content-Length cannot be modified.   

      
The modify condition of a header is saved at /svc/{virtual host name}/headers.txt. 
The header allows multiple configurations as long as each fits to the condition. 
All modifications will be applied at the same time. 

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
    
IP, GeoIP, Header and URL forms can be used for a {Match} configuration.

-  **IP**
   $IP[...] format is used and supports IP, IP Range, Bitmask and Subnet formats.

-  **GeoIP**
   $IP[...] format is used and :ref:`access-control-geoip` must be predefined.
   Country codes of `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ and `ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ are supported.
     
-  **Header**
   $HEADER[Key : Value] format is used. 
   Value supports specific expressions and patterns. 
   When the ``Value`` is omitted, the existence of the header that is corresponding to the ``Key`` will be the condition used to make a decision.
    
-  **URL**
   $URL[...] format is used and can be omitted. Specific expressions and patterns will be recognized.
    
{$REQ} and {$RES} configure how to modify the header.
Generally ``set`` and ``append`` use {Key: Value} for configuration, and if the Value is omitted, an empty value("") will be inserted. 
For ``unset``, only the {Key} value is required.

``set`` , ``unset`` , ``append`` can be used for {Action}.

-  ``set``  Adds the Key and Value that are defined in the request/response header to the header. 
   If an identical Key exists, set overwrites the previous value.    

-  ``put``  This is similar to ``set``. Adds the Key and Value defined in the request/response header to the header. 
   Unlike ``set``, put inserts an extra set of the Key and Value.

-  ``unset`` Discards the header related to the Key that is defined in the request/response header

-  ``append``  This is similar to ``set``, while this setting uses a comma(,) to combine the previous Value with the configured Value when a related Key is found.

{Condition} identifies response code families--such as 2xx, 3xx, 4xx and 5xx--instead of specific response codes--such as 200 and 304.
If {Condition} is not matching, a modification will not be reflected, even if {Match} is matching.
If {Condition} is omitted, a response code will not be examined.


Original Header
---------------------

In order to maintain decent performance, only a standard header will be selectively identified from headers that the origin server transmits. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <OriginalHeader>OFF</OriginalHeader>   
    
-  ``<OriginalHeader>``

   -  ``OFF (default)`` Ignores if the headers are not a standard. 
   
   -  ``OFF``  Saves non-standard headers and transfers them to the client.
      However, this option will consume more memory and storage.


Via Header
---------------------

This section explains how to configure HTTP responses that will be sent to a client that either specify the Via header or not. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ViaHeader>ON</ViaHeader>   
    
-  ``<ViaHeader>``
    
   - ``ON (default)`` Specifies the Via header, as shown below.
     ::
      
        Via: STON/2.0.0
   
   - ``OFF``  Omits the Via header
   
   
Server Header
---------------------
 
This section explains how to configure HTTP responses that will be sent to the client that either specifies the Server header or not. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ServerHeader>ON</ServerHeader>   
    
-  ``<ServerHeader>``
    
   -  ``ON (default)`` Specifies the Server header of the origin server ::
   
   -  ``OFF``  Omits the Server header



URL Preprocessing
====================================

`Regular expression <http://en.wikipedia.org/wiki/Regular_expression>`_ is used to modify requested URLs. 
If URL preprocessing is defined, all client requests(HTTP or File I/O) should pass through the URL Rewriter.

.. figure:: img/urlrewrite1.png
   :align: center
      
   After passing through the URL Rewriter, the virtual host can be accessed.
   
If an approaching Host name has been modified by the URL Rewriter, it is considered that the clinet's HTTP request Host header has been modified.
URL preprocessing is configured at the virtual host setting(vhosts.xml).
Most configurations are subordinated to the virtual host, but URL preprocessing can modify the Host name that a client requested, so it should be configured in the same block as the virtual host. ::

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
   The ``AccessLog (default: Replace)`` attribute configures URLs that will be recorded in the Access log. 
   ``Replace`` records preprocessed URLs (/logo.jpg), whereas ``Pattern`` records unprocessed URLs (/baseball/logo.jpg) to the Access log.
   
   -  ``<Pattern>`` configures patterns to be matched. 
      A single pattern is expressed with parenthesis (e.g. ( )).
   
   -  ``<Replace>`` configures convert format. 
      Identical patterns can use expressions like #1 and #2. #0 stands for the entire URL that is requested. 
      A maximum of nine patternes (#9) can be configured.
      
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


.. _handling_http_requests_compression:

Compression
====================================
Cached content is deliverable in compression.
Content files MUST be categorized by :ref:`caching-policy-accept-encoding`  ::

   Accept-Encoding: gzip, deflate

.. figure:: img/compression_1.png
   :align: center
      
   Delivered in on-the-fly compression.

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Compression Method="gzip" Level="6" SourceSize="2-2048">OFF</Compression>

-  ``<Compression>``

   -  ``OFF (default)`` no compression
   
   -  ``ON`` compressed by the following attributes

      -  ``Method (default: gzip)`` Compression method - only gzip supported for now.
      -  ``Level (default: 6)`` Compression level - dependant on ``Method``, and 1~9 for gzip. A higher value means slower and more compression. A lower one means faster and less compression.
      -  ``SourceSize (default: 2-2048, unit: KB)`` Source size in range. 
         Too small files might be hardly compressed. 
         Too large files might consume too much CPU, on the other hand.

Compressed content is cached and stored separately from its original. More requests for the same content do not incur compression. 
The list of files to compress is configurable in /svc/{vhost}/compression.txt in an orderly manner. ::
   # /svc/www.example.com/compression.txt
   # Separated by commas ( , ) .
   # Formatted in {URL Condition}, {Method}, {Level}

   /sample.css, no       // No compression
   *.css                 // Compress *.css with default method and level
   *.htm, gzip           // Compress *.htm with gzip (default level)
   *.xml, , 9            // Compress *.xml with level 9 (default method)
   *.js, gzip, 5         // Compress *.js with gzip level 5.

Compression consumes CPU resource highly.
The following is a test result from gzip level 9.

-  ``OS`` CentOS 6.3 (Linux version 2.6.32-279.el6.x86_64 (mockbuild@c6b9.bsys.dev.centos.org) (gcc version 4.4.6 20120305(Red Hat 4.4.6-4) (GCC) ) #1 SMP Fri Jun 22 12:19:21 UTC 2012)
-  ``CPU`` `Intel(R) Xeon(R) CPU E5-2603 0 @ 1.80GHz (8 processors) <http://www.cpubenchmark.net/cpu.php?cpu=Intel%20Xeon%20E5-2603%20@%201.80GHz>`_
-  ``RAM`` 8GB
-  ``HDD`` SAS 275GB X 5EA

======================= ============== ======== ============== ========================= =====================
Size                    Comp. Ratio(%)  Files    Latency (ms)   Client Traffic (Mbps)    Origin Traffic (Mbps)
======================= ============== ======== ============== ========================= =====================
1KB                     26.25          5288     6.72           40.58                     55.02
2KB                     57.45          5238     7.20           41.52                     97.58
4KB                     76.94          5236     7.18           42.44                     184.04
8KB                     87.61          5021     7.53           41.87                     337.80
16KB                    93.32          4608     8.30           41.19                     616.83
32KB                    96.26          3495     13.55          34.53                     924.22
64KB                    97.79          1783     24.50          20.71                     938.83
bootstrap.css(20KB)     86.87          3944     9.67           83.79                     638.25
bootstrap.min.js(36KB)  73.00          1791     51.50          139.00                    514.86
======================= ============== ======== ============== ========================= =====================

If ``<Compression>`` is turned on, uncompressed files are requested from origin servers, meaning reseponses from no Accept-Encoding headers.
A Content-Encoding header means compressed from the origin server, and STON does not compress the content again.


.. note::

   If any pre-compresed content undergoes ``<Compression>``, this might cause a serious problem. Please keep in mind the followings.

   1. Compress new content.
   2. Do not compress content which is pre-compressed in its origin server.
   3. Invalidate AND compress content which is not compressed in its origin server. 
