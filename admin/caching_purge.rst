.. _caching-purge:

Chapter 5. Caching Invalidation
******************

This chapter explains how to invalidate cached content.
Detailed invalidation APIs are provided for different conditions.

Cached content has a lifetime based on :ref:`ttl-time-to-live`.
However, if content is updated and the changes must be effective immediately, it is unnecessary to wait until :ref:`ttl-time-to-live` is over.
`Purge`_ / `Expire`_ / `HardPurge`_ will immediately invalidate or 'purge' content.

The purge API can be called by the browser, but mostly it is automated.
In an FTP file upload, for instance, as soon as the upload is completed, `Purge`_ is called immediately.
Policies can be configured as shown below. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Purge2Expire>NONE</Purge2Expire>
   <RootPurgeExpire>ON</RootPurgeExpire>
   <ResCodeNoCtrlTarget>200</ResCodeNoCtrlTarget>   

-  ``<Purge2Expire> (default: NONE)``

   This is a fool-proof feature. A `Purge`_ request can be processed with `Expire`_ depends on the configuration.
   For example, when `Purge`_ the root directory(/), this can create excessive load on the origin server by purging large amount of contents.
   In cases like this, `Expire`_ will prevent excessive load on the origin server.

   - ``NONE`` Does not use `Expire`_.
   - ``ROOT`` Uses `Expire`_ for `Purge`_ of domain root directory(/).
   - ``PATTERN`` Uses `Expire`_ for pattern `Purge`_.
   - ``ALL`` Uses `Expire`_ for all `Purge`_.

-  ``<RootPurgeExpire> (default: ON)``
   
   Unintended `Purge`_ / `Expire`_ of root directory(/) might occur excessive load on the origin server.
   This setting can protect `Purge`_ / `Expire`_ of root directory.
   The setting has a priority over ``<Purge2Expire>``.

   - ``ON`` Allows `Purge`_ / `Expire`_.
   - ``PURGE`` Only allows `Purge`_.
   - ``EXPIRE`` Only allows `Expire`_.
   - ``OFF`` Prohibits all `Purge`_ / `Expire`_.

-  ``<ResCodeNoCtrlTarget> (default: 200)``

   Configures the HTTP response code when target objects are missing for `Purge`_ , `Expire`_ , `HardPurge`_ , or `ExpireAfter`_.


Targets may be called by either URL or pattern. ::

   example.com/logo.jpg      // URL
   example.com/img/          // URL
   example.com/img/*.jpg     // pattern
   example.com/img/*         // pattern
   
Other than some specific URLs, a patterned URL may indicate contents to purge as well. (e.g. *.jpg)
However, the number of targeted item is remain uncertain, until the command is executed.
For this reason, administrator might make an unintended call to purge too many targets, which consumes too much CPU resource and causes breakdown.

Therefore, it is strongly recommended to use a specific URL while the service is running.
A patterned URL or directory representation is only used as an administrative purpose when the service is not running.


.. note::

   Access to particular directories such as example.com/files/ is forbidden for security and returns 403 FORBIDDEN.  
   However the root directory is an exempt.  
   For an example, a client accesses example.com and his browser requests the root directory (/). ::
   
      GET / HTTP/1.1
      Host: example.com
   
   The web server returns the default page. (possible index.html or index.htm)
   A page is returned for the root directory for most web services.
   
   However 200 OK page is returned a cache server for the root directory (/).
   The cache server may not be aware which page is returned.
   The cache server interpretes a directory just as a URL. ::
   
      example.com/img/          // example.com The returned page for /img/ access
      example.com/              // example.com The default page (/) from the virtual host
      example.com/img/*         // example.com /img directory and its subdirectories from the virtual host
      example.com/*             // example.com All contents from the virtual host
         



.. toctree::
   :maxdepth: 2


.. _api-cmd-purge:

Purge
====================================

Invalidates target content to induce downloading the content from the origin server.
Content will be cached when accessing the content for the first time after a Purge.
If content is not available from the origin server due to error situations, STON retrieves invalidated content in order to keep the service continually available.
Retrieved content is renewed after the time set by ConnectTimeout. ::

    http://127.0.0.1:10040/command/purge?url=...
    
Target content can be designated by a URL, directory, or pattern.
Multiple targets from multiple domains can also be designated by using a "|"(vertical bar) identifier.
If a domain name is omitted, the most recently used domain name will be used. ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
The result will be returned in JSON format. 
The number of target contents, size and elapsed time (units in millisecond) are returned.
Purged contents cannot be purged again. ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }
    
``<Purge2Expire>`` can set Expire as a substitute for a specific conditioned Purge.
``<ResCodeNoCtrlTarget>`` can be used to set the HTTP response code for the reply without a result.

.. note::
   
   If all origin servers are excluded due to errors, Purge will not work because content cannot be updated.


.. _api-cmd-expire:
   
Expire
====================================

Immediately expires the TTL of the targeted content.
Check modifications from the origin server when the content is accessed for the first time after expiration. 
If content has not been modified, the TTL will be prolonged without downloading the content. ::

    http://127.0.0.1:10040/command/purge?url=...
    
Except the above functions, Expire works the same as `Purge`_.


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

Set the TTL expiration time of targeted contents to the entered period (in seconds) from the moment of an API call.
The ExpireAfter command can advance the expiration time so that content can be renewed earlier, 
or it can reduce the load on the origin server by extending the expiration time. :: 

   http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

The function call format is similar to `Purge`_ / `Expire`_ but the TTL expiration time can be set with the ``sec`` parameter(in seconds).
If the ``sec`` paramter is omitted, the default value of 1 day(86400 seconds) is applied, and setting it to 0 will not be allowed. 
The result is identical to those of `Purge`_ / `Expire`_, except ExpireAfter works regardless of origin server errors. 
The HTTP response code for a reply without a result can be configured with the ``<ResCodeNoCtrlTarget>``.

.. note::
   ExpireAfter is not an API that configures the custom TTL or default TTL; rather, it only sets the expiration time of cached content. Content that is cached after an ExpireAfter call will not be affected.

   
   It is recommended to enter the ``sec`` parameter before the ``url`` parameter, otherwise the ``sec`` parameter could be recognized as a query string of the ``url`` parameter.

   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ APIs can retrieve contents when the origin server is down. 
On the other hand, a HardPurge permanently discards content. 
If you use a HardPurge to erase content, it cannot be serviced when the origin server is out of service. 
The HTTP response code for the reply without a result can be configured with the ``<ResCodeNoCtrlTarget>``

    http://127.0.0.1:10040/command/hardpurge?url=...


HTTP Method
====================================

An API invalidation can be called by an expanded HTTP Method. ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
An HTTP Method basically works under the Manager port and the service port(80). 
It can be set in the :ref:`env-host` of the HTTP Method requested via the service port.


.. _api-etc-post:

POST Standard
====================================

An API invalidation can be called by POST as shown below. ::

   POST /command/purge HTTP/1.1
   Content-Length: 37
 
   url=http://ston.winesoft.co.kr/sample.dat
    
 
