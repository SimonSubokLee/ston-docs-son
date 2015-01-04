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
The HTTP response code for the reply without a result can be configured with the ``<ResCodeNoCtrlTarget>``

.. note::
   ExpireAfter command only set the expiration time of cached contents
   ExpireAfter는 캐싱되어있는 컨텐츠의 현재 만료시간만을 설정할 뿐 커스텀TTL이나 
   설정된 기본 TTL을 변경시키는 API가 아니다. 
   ExpireAfter 호출뒤에 캐싱된 컨텐츠들은 영향을 받지 않는다.
   
   
   url파라미터를 먼저 입력하는 경우 sec파라미터가 url파라미터의 QueryString으로 인식될 수 있다. 
   그러므로 sec파라미터가 먼저 입력되는 것이 안전하다.
   
   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ 이상의 API는 원본서버 장애상황에서도 컨텐츠가 
사라지지 않고 정상적으로 동작한다. 
하지만 HardPurge는 컨텐츠의 완전한 삭제를 의미한다. 
HardPurge는 가장 강력한 삭제방법이지만 삭제한 컨텐츠는 원본서버에 장애가 발생해도 되살릴 수 없다. 
결과없는 응답에 대해서는 ``<ResCodeNoCtrlTarget>`` 로 HTTP 응답코드를 설정할 수 있다. ::

    http://127.0.0.1:10040/command/hardpurge?url=...


HTTP Method
====================================

무효화 API를 확장 HTTP Method로 호출할 수 있다. ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
HTTP Method는 기본적으로 Manager포트와 서비스(80)포트에서 동작한다. 
서비스포트로 요청되는 HTTP Method의 :ref:`env-host` 에서 설정한다.


.. _api-etc-post:

POST 규격
====================================

무효화 API를 다음과 같이 POST로 호출할 수 있다. ::

   POST /command/purge HTTP/1.1
   Content-Length: 37
 
   url=http://ston.winesoft.co.kr/sample.dat
    

