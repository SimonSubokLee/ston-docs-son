.. _caching-purge:

Chapter 5. Caching Purge
******************

This chapter explains how to purge cahced contents.
Due to various environments and conditions, specified APIs are required.

Contents usually have a renewal period based on :ref:`ttl-time-to-live`.
However, if contents are obviously modified and you want to apply the changes immediately, you don't have to wait until :ref:`ttl-time-to-live` is expired.
`Purge`_ / `Expire`_ / `HardPurge`_ will immediately purge contents.

The purge API can be simply called by the browser, but mostly it is automated.
FTP file upload, for instance, as soon as upload is completed, `Purge`_ is called right away.
Administrator can configure a few policies as belows. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Purge2Expire>NONE</Purge2Expire>
   <RootPurgeExpire>ON</RootPurgeExpire>
   <ResCodeNoCtrlTarget>200</ResCodeNoCtrlTarget>   

-  ``<Purge2Expire> (default: NONE)``

   `Purge`_ request is processed with `Expire`_ depends on the configuration.
   For example, when `Purge`_ the root directory(/), this can create excessive load on the origin server by purging large amount of contents.
   In case like this, `Expire`_ will prevent excessive load on the origin server.

   - ``NONE`` Does not use `Expire`_.
   - ``ROOT`` Uses `Expire`_ for `Purge`_ of domain root directory(/).
   - ``DIR`` Uses `Expire`_ for `Purge`_ of all directories.
   - ``PATTERN`` Uses `Expire`_ for pattern `Purge`_.
   - ``MULTIPLE`` Uses `Expire`_ for directory and pattern `Purge`_.
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

   Configures the HTTP response code when target objects are missing for `Purge`_ , `Expire`_ , `HardPurge`_ , `ExpireAfter`_.


.. warning:

   Other than specific URLs, pattern or directory can be purged as well.
   However, until the command is performed, number of target item is unclear.
   For this reason, administrator might select too many targets without intention, and it will occupy too much CPU resource and causes performance drop.

   Therefore, it is strongly recommended to use a specific URL during the service.
   Pattern or directory representation is only used as an administrative purpose when the service is not running.


.. toctree::
   :maxdepth: 2


.. _api-cmd-purge:

Purge
====================================

Invalidates target contents to induce download the contents from the origin server.
Contents will be cached when accessing the contents for the first time after Purge.
If contents are not available from the origin server due to error situations, the STON retrieves invalidated contents in order to keep the service available all the time.
Retrieved contents are renewed after the time set by ConnectTimeout. ::

    http://127.0.0.1:10040/command/purge?url=...
    
The target contents can be designated by URL, directory, pattern.
Multiple targets from multiple domains can also be designated by using "|"(vertical bar) identifier.
If domain name is omitted, most recently used domain name will be used. ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
The result will be returned in JSON format. 
The number of target contents, size and elapsed time(unit in millisecond) are returned.
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
   
   If all origin servers are excluded due to errors, Purge will not work because contents cannot be updated.


.. _api-cmd-expire:
   
Expire
====================================

Immediately expires TTL of the targeted contents.
Check modification from the origin server when the contents are accessed for the first time after expiration. 
If the contents have not been modified, TTL will be prolonged without downloading the contents. ::

    http://127.0.0.1:10040/command/purge?url=...
    
Except the above functions, Expire works same as `Purge`_.


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

Set the TTL expiration time of targeted contents to entered period(in second) from the moment of API call.
ExpireAfter command can advance expiration time so the contents can be renewed earlier, 
or reduce the load of origin server by extending expiration time. :: 

   http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

The function call format is similar to `Purge`_ / `Expire`_ but TTL expiration time can be set with the ``sec`` parameter(in second).
If the ``sec`` paramter is omitted, default value of 1 day(86400 seconds) is applied, and setting it to 0 will not be allowed. 
The result is identical to those of `Purge`_ / `Expire`_, except ExpireAfter works regardless of error of the origin server. 
The HTTP response code for the reply without a result can be configured with the ``<ResCodeNoCtrlTarget>``.

.. note::
   ExpireAfter is not an API that configures the custom TTL or default TTL, but only set the expiration time of cached contents. 
   Contents that are cached after ExpireAfter call will not be affected.

   
   It is recommended to enter the ``sec`` parameter before the ``url`` parameter, othewise the ``sec`` parameter could be recognized as a QueryString of the ``url`` parameter.

   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ APIs can retrieve contents when the origin server is down. 
On the other hand, HardPurge permanently discard contents. 
If you use HardPurge to erase contents, they cannot be serviced when the origin server is out of service. 
The HTTP response code for the reply without a result can be configured with the ``<ResCodeNoCtrlTarget>``

    http://127.0.0.1:10040/command/hardpurge?url=...


HTTP Method
====================================

Invalidating API can be called by expanded HTTP Method. ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
HTTP Method basically works under the Manager port and the service port(80). 
It can be set in the :ref:`env-host` of the HTTP Method requested via the service port.


.. _api-etc-post:

POST Standard
====================================

Invalidating API can be called by POST as below. ::

   POST /command/purge HTTP/1.1
   Content-Length: 37
 
   url=http://ston.winesoft.co.kr/sample.dat
    

