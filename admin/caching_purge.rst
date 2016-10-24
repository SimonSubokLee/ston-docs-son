.. _caching-purge:

Chapter 5. Content Purge
*******************************

This chapter will explain how to purge cached content. Because there are many different conditions and environments, many different parts of the API are necessary.

Content cached from the origin server have update cycles based on the :ref:`caching-policy-ttl`. However, if the administrator wishes to immediately have the changes be effective, there is no need to wait until the :ref:`caching-policy-ttl` expires. By using `Purge`_/`Expire`_/`HardPurge`_, content can immediately be purged.

The purge API can be called by the browser, but in most cases it is automated. For example, when an FTP file upload is completed, `Purge`_ is called immediately. Administrators can configure behaviors in several ways as shown below. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Purge2Expire>NONE</Purge2Expire>
   <RootPurgeExpire>ON</RootPurgeExpire>
   <ResCodeNoCtrlTarget>200</ResCodeNoCtrlTarget>

-  ``<Purge2Expire> (default: NONE)``

   `Purge`_ requests will be processed as `Expire`_ depending on the setting. For example, if a pattern (\*.jpg) used alongside `Purge`_ , the deletion of an unexpectedly large amount of content can cause immense load on the origin server. In this case, if all the requests are processed as `Expire`_ requests, that server load can be prevented.
   
   - ``NONE`` Does not process requests as `Expire`_.
   - ``ROOT`` Processes requests for only the root directory (/) as `Expire`_ .
   - ``PATTERN`` Processes requests with patterns as `Expire`_.
   - ``ALL`` Processes all requests as `Expire`_.

-  ``<RootPurgeExpire> (default: ON)``
   
   An unintentional `Purge`_/`Expire`_ request on the root directory can also cause load on the origin server. This setting can intercept `Purge`_/`Expire`_ requests and prevent them. This setting takes precedence over ``<Purge2Expire>``.
   
   - ``ON`` `Purge`_/`Expire`_ is allowed.
   - ``PURGE`` Only `Purge`_ is allowed.
   - ``EXPIRE`` Only `Expire`_ is allowed.
   - ``OFF`` All `Purge`_/`Expire`_ requests are prevented.

-  ``<ResCodeNoCtrlTarget> (default: 200)``

   Sets the HTTP response code for when `Purge`_, `Expire`_, `HardPurge`_, and `ExpireAfter`_ have no target object.
   

Targets can be either URLs or patterns. ::

   example.com/logo.jpg      // URL
   example.com/img/          // URL
   example.com/img/*.jpg     // pattern
   example.com/img/*         // pattern
   
While patterned URLs can be called, the amount of actual content being targeted cannot be known until the command is executed. Therefore, the administrator may underestimate the amount of content and try to purge too many targets, consuming more CPU than expected and causing strain on the system.
   
As such, it is strongly recommended to use only specific URLs. Patterned representations should only be used for the sake of administrative purposes when the service is not running.


.. note::

   For security reasons, accessing specific directories (e.g. example.com/files/) is forbidden and returns 403 FORBIDDEN. However, the root directory is exempt: that is, if a user accesses example.com, their browser will request the root directory (/). ::
   
      GET / HTTP/1.1
      Host: example.com
   
   The web server will respond with the default page set by the administrator (e.g. index.html). Most web services will return a page, not a directory, for the root directory (/).
   
   However, when the cache server accesses the root directory, it will think it has received a 200 OK page, and won't even be able to know what page was returned. In other words, to the cache server, a directory is just another URL. ::
   
      example.com/img/          // The resulting page from accessing /img/ on the example.com virtual host
      example.com/              // The default page (/) for the example.com virtual host
      example.com/img/*         // The /img/ directory and all pages below it on the example.com virtual host
      example.com/*             // All content on the example.com virtual host
         


.. toctree::
   :maxdepth: 2


.. _api-cmd-purge:

Purge
====================================

Purges the target in order to have it be redownloaded from the origin server. Content will be cached again when it is first accessed after a purge. If the content is not available on the origin server due to an error, the purged content will be restored to keep the service running. This restored content will be updated after the time set by ConnectTimeout. ::

    http://127.0.0.1:10040/command/purge?url=...
    
Target content can be designated with URLs and patterns, and can also be designated with vertical bars ("|") to indicate multiple domains and multiple targets. If the domain name is omitted, the most recently used domain name is used. ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
The results are returned in JSON format. The number and size of purged items, as well as the elapsed time (units: ms) will be displayed. Content that has already been purged will not be purged again. ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }
    
Using the ``<Purge2Expire>`` tag, Purge can be set to Expire under certain conditions. For a response with no results, the HTTP response code can be set with ``<ResCodeNoCtrlTarget>``. 

.. note::
   
   If all origin servers are down due to errors, Purge will not work, as content is unable to be updated.
   

.. _api-cmd-expire:
   
Expire
====================================

The TTL of the target content is set to expire immediately. A check for modification is made when the content is first accessed after expiring. If there is no change, there is no redownload; only the TTL is extended. ::

    http://127.0.0.1:10040/command/expire?url=...
    
Everything else is identical to `Purge`_.


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

The TTL of the target content is set so that the content expires the input number of seconds after the API is called. ExpireAfter can make the expiration time earlier and make content update faster, or it can make the expiration time later and reduce load on the origin server. ::

   http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

Though the function call format resembles `Purge`_ and `Expire`_, the sec parameter (in seconds) can also set the expiration date. If the sec parameter is omitted, the default value of 1 day (86400 s) is applied, and setting it to 0 is not allowed. For a response with no results, the HTTP response code can be set with ``<ResCodeNoCtrlTarget>``. 

.. note::

   ExpireAfter only sets the current expiration time, and does not affect custom TTLs or default TTLs. There is no change to cached content after an ExpireAfter call.   
   
   If the url parameter is entered first, the sec parameter may be recognized as a query string of the url parameter. Therefore, it is recommended to set enter the sec parameter first.
   
   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

If there is an error in the origin server, `Purge`_/`Expire`_/`ExpireAfter`_ will retain the content and continue normally. In contrast, HardPurge means the content is permanently deleted. As HardPurge is the most powerful deletion method, deleted content cannot be restored if there is an error. For a response with no results, the HTTP response code can be set with ``<ResCodeNoCtrlTarget>``. ::

    http://127.0.0.1:10040/command/hardpurge?url=...


Default Purge Behavior
====================================

The behavior of content restoration after a Purge API call can be configured. ::

   # server.xml - <Server><Cache>
   
   <Purge>Normal</Purge>
      
-  ``<Purge>`` 
   
   - ``Normal (default)`` Behaves as if it was `Purge`_. (Content is restored if there is an error.)
   
   - ``Hard`` Behaves as if it was `HardPurge`_. (Content is not restored if there is an error.)

.. _api-etc-httpmethod:
   
HTTP Method
====================================

The purge API can be called with an extended HTTP Method. ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
HTTP methods fundamentally work under the manager port and the service port (80). HTTP Method requests sent to the service port can be configured in :ref:`env-host`.


.. _api-etc-post:

POST Standard
====================================

The purge API can be called with POST, as shown below. ::

   POST /command/purge HTTP/1.1
   Content-Length: 37
 
   url=http://ston.winesoft.co.kr/sample.dat
    

