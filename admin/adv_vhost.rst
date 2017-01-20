.. _adv-vhost:

Chapter 14. Advanced Virtual Host Configuration
******************************************************************

This chapter will discuss advanced topics related to virtual host configuration.

Each virtual host is supposed to match its origin. (a domain or an IP) But sometimes a virtual host has to be connected to others, or the other way round. Depending on functionalities,  :ref:`monitoring_stats_vhost_client` / :ref:`admin-log-access` policies might be different.


.. toctree::
   :maxdepth: 2


.. _adv-vhost-url-rewrite:


URL Preprocessing
====================================

`Regular expressions <http://en.wikipedia.org/wiki/Regular_expression>`_ are used to modify the requested URLs. If URL preprocessing is defined, all client requests (HTTP or File I/O) must pass through the URL Rewriter.

.. figure:: img/urlrewrite1.png
   :align: center
      
   The request can only reach the virtual host by passing through the URL Rewriter.
   
If an approaching Host name is modified by the URL Rewriter, then it will consider it as if the Host header was modified by the client's HTTP request. URL preprocessing is configured in the virtual host settings (vhosts.xml). While most settings are under the virtual host, URL preprocessing can change the name of the Host requested by the client, so the settings must be on the same level as the virtual host. ::

   # vhosts.xml
   
   <Vhosts>
      <Vhost ...> ... </Vhost>
      <Vhost ...> ... </Vhost>
      <URLRewrite ...> ... </URLRewrite>
      <URLRewrite ...> ... </URLRewrite>
   </Vhosts>
    
Multiple configurations are allowed, and the regular expressions will be checked in order. ::

   # vhosts.xml - <Vhosts>
   
   <URLRewrite AccessLog="Replace">
       <Pattern>www.example.com/([^/]+)/(.*)</Pattern>
       <Replace>#1.example.com/#2</Replace>
   </URLRewrite>
    
-  ``<URLRewrite>``

   Configures URL preprocessing.
   ``AccessLog (default: Replace)`` Configures URLs that will be recorded in the Access log. ``Replace`` records URLs after processing (/logo.jpg), while ``Pattern`` records URLs after processing (/baseball/logo.jpg).
   
   -  ``<Pattern>`` Configures the patterns to be matched. A single pattern is expressed with parentheses ().
   
   -  ``<Replace>`` Configures the conversion format. Patterns that match can be used with expressions like #1 and #2. #0 stands for the entire requested URL. A maximum of nine patterns (up to #9) can be configured.
      
Throughput is provided by :ref:`monitoring_stats` and can also be checked via :ref:`api-graph-urlrewrite`. URL preprocessing can work alongside :ref:`media-trimming` and :ref:`media-hls` to simplify expressions further. ::

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
    
If one of the five special XML characters are used, then the pattern must be surrounded with a <![CDATA[ ... ]]> tag. If configured using :ref:`wm`, all patterns are processed as CDATA.




.. _adv-vhost-facadevhost:

Facade Virtual Host
====================================

Because ``<Alias>`` is just a nickname for the virtual host, it will not provide separate statistics and logs. If you want to use the same virtual host but obtain different :ref:`monitoring_stats_vhost_client` and :ref:`admin-log-access`  depending on the domain, a Facade Virtual Host can be configured. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Name="example.com">
       ...
    </Vhost>
    
    <Vhost Name="another.com" Status="facade:example.com">
       ...
    </Vhost>

This can be done by inputting ``facade:`` + ``virtual host`` into the ``Status`` property. In the previous example, the  :ref:`monitoring_stats_vhost_client` and :ref:`admin-log-access` will be recorded for clients that request another.com, not example.com.


.. _adv-vhost-sub-path:
    
Sub-Path
====================================

A single virtual host can have different sub-paths. These sub-paths can be configured to be handled by separate virtual hosts. ::

   # vhosts.xml - <Vhosts>
   
   <Vhost Name="sports.com">
     <Sub Status="Active">
       <Path Vhost="baseball.com">/baseball/<Path>
       <Path Vhost="football.com">/football/<Path>
       <Path Vhost="photo.com">/*.jpg<Path>
     </Sub>
   </Vhost>

   <Vhost Name="baseball.com" />
   <Vhost Name="football.com" />
   <Vhost Name="photo.com" />

-  If the page path or pattern matches the ``<Sub>`` input, then it will be sent to the corresponding virtual host. If they do not match, then the page will be handled by the current virtual host.
    
   - ``Status (default: Active)`` Sub-paths are ignored when inactive.

   -  ``<Path>`` If the URI requested by the client and the path match, the request will be sent to ``Vhost``. Only paths or patterns are allowed. ::
      
         <Path Vhost="baseball.com">baseball<Path>
         <Path Vhost="photo.com">*.jpg<Path>
      
      If input as above, they will be parsed as /baseball/ and /\*.jpg, respectively.

For example, if the client requests the following, the request will be sent to the football.com virtual host. ::

   GET /football/rank.html HTTP/1.1
   Host: sports.com


.. _adv-vhost-redirection-trace:

Redirect Tracing
====================================

When the origin server responds with the Redirect responses (301, 302, 303, 307), the location header is tracked to request the content.

   .. figure:: img/conf_redirectiontrace.png
      :align: center

      The redirection is hidden from the client.

::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <RedirectionTrace>OFF</RedirectionTrace>

-  ``<RedirectionTrace>``

   - ``OFF (default)`` Saved as 3xx responses.

   - ``ON`` Downloads content from the address specified in the location header.
      Works only if matched to the format or with a valid Location header.
      Traced only once to prevent infinite redirects.


.. _adv-vhost-link:

Virtual Host Link
====================================

Even if the content is distributed across multiple origins, the service is still operable as if the content were integrated using virtual host links.
It is particularly useful in environments where content is scattered due to on-premise to cloud storage migration, storage capacity, and cost.

.. figure:: img/adv_vhost_link.png
   :align: center

   Content missing from cloud.com is delivered by nas.com.

::

   # vhosts.xml - <Vhosts><Vhost>

   <VhostLink Condition="...">...</VhostLink>

-  ``<VhostLink>`` The virtual host name to delegate requests to. If the original response to the content satisfies `` Condition``, the request is delegated to the specified virtual host. Only one can be set.

   - ``Condition`` HTTP response code / pattern (1xx, 2xx, 3xx, 4xx, 5xx), fail (If failed to cache from source)

Even if the client request is delegated to another virtual host, the: ref: `monitoring_stats_vhost_client` and: ref:` admin-log-access` are recorded in the virtual host accessed by the client.

.. note::

    Please note that if the virtual hosts’ configurations in the link are different from each other, it may operate in unintentional ways.
    If the virtual host link is concatenated with A (simple caching) -> B (image compression)
    Images processed in A are not compressed, but images processed in B are compressed.

For example, if you are moving content from nas.com to cloud.com, you can only send requests to nas.com for content not on cloud.com (= 404 Not Found).
In the following cases: ref: `monitoring_stats_vhost_client` and: ref:` admin-log-access` are recorded on cloud.com, even if the request is handled by nas.com.

::

   # vhosts.xml - <Vhosts>

   // Content not on cloud.com (= 404 Not Found) will be served on nas.com.
   <Vhost Name="cloud.com">
     <VhostLink Condition="404">nas.com</VhostLink>
   </Vhost>

   <Vhost Name="nas.com">
   </Vhost>


The vhostlink field of: ref: `admin-log-access` will tell you which virtual host the client request was processed on.
"-" means the request is not linked, and "nas.com" means that the request has been linked and processed at nas.com. ::

    #Fields: date time s-ip cs-method cs-uri-stem …(…)… vhostlink
    2016.11.24 16:52:24 220.134.10.5 GET /web/h.gif …(…)… -
    2016.11.24 16:52:26 220.134.10.5 GET /favicon.ico …(…)… nas.com

If the link has occurred multiple times, all virtual hosts linked with "+" delimiters are specified.
In this case, the last virtual host is the virtual host that processed the last request.

You can link multiple virtual hosts to different conditions as follows:

::

   # vhosts.xml - <Vhosts>

  // When the origin server responds with 5xx or fails to cache (= fail), it delegates the request to bar.com.
   <Vhost Name="foo.com">
     <VhostLink Condition="5xx,fail">bar.com</VhostLink>
   </Vhost>

   // Delegates the request to helloworld.com when the origin server responds with 4xx.
   <Vhost Name="bar.com">
     <VhostLink Condition="4xx">helloworld.com</VhostLink>
   </Vhost>

   // When the origin server responds with 403, 404, or 5xx, it delegates the request to example.com.
   <Vhost Name="helloworld.com">
     <VhostLink Condition="403,404,5xx">example.com</VhostLink>
   </Vhost>

   // Does not delegate any more.
   <Vhost Name="example.com">
   </Vhost>

.. figure:: img/adv_vhost_link_worst.png
   :align: center

   It might look akward but not impossible.

In the above example, the: ref: `admin-log-access` of foo.com looks like this: ::

   #Fields: date time s-ip cs-method cs-uri-stem …(…)… vhostlink
   2016.11.24 16:52:24 220.134.10.5 GET /test.jpg …(…)… bar.com+helloworld.com+example.com

In the following cases, the link is immediately terminated.

* If the target virtual host does not exist (foo.com ->?)
* If you specified yourself as the destination virtual host (foo.com -> foo.com)
* If a recursive link occurs (foo.com -> bar.com -> foo.com)
