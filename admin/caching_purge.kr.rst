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

타겟 컨텐츠를 무효화시켜 원본서버로부터 컨텐츠를 다시 다운로드 받도록 한다. 
Purge후 최초 접근 시점에 원본서버로부터 컨텐츠를 다시 캐싱한다. 
만약 원본서버에 장애가 발생하여 컨텐츠를 가져올 수 없다면 무효화된 컨텐츠를 다시 복원시켜 
서비스에 장애가 없도록 처리한다. 
이렇게 복원된 컨텐츠는 해당 시점으로부터 ConnectTimeout설정만큼 뒤에 갱신한다. ::

    http://127.0.0.1:10040/command/purge?url=...
    
타겟 컨텐츠는 URL, 디렉토리, 패턴으로 지정할 수 있을 뿐만 아니라 "|"(Vertical Bar)를 
구분자를 사용하여 복수의 도메인에 복수의 타겟을 지정할 수 있다. 
만약 도메인 이름이 생략되었다면 최근 사용된 도메인을 사용한다. ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
결과는 JSON형식으로 제공된다. 
타겟 컨텐츠 개수/용량 및 처리시간(단위: ms)이 명시된다. 
이미 Purge 된 컨텐츠는 다시 Purge되지 않는다. ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }
    
``<Purge2Expire>`` 를 통해 특정조건의 Purge를 Expire로 동작하도록 설정할 수 있다.
결과없는 응답에 대해서는 ``<ResCodeNoCtrlTarget>`` 로 HTTP 응답코드를 설정할 수 있다.

.. note::
   
   원본서버가 장애로 인해 모두 배제되었다면 컨텐츠를 갱신할 수 없기 때문에 Purge가 동작하지 않는다.
   

.. _api-cmd-expire:
   
Expire
====================================

타겟 컨텐츠의 TTL을 즉시 만료시킨다. 
Expire후 최초 접근 시점에 원본서버로부터 변경여부를 확인한다. 
변경되지 않았다면 TTL연장만 있을 뿐 컨텐츠 다운로드는 발생하지 않는다. ::

    http://127.0.0.1:10040/command/purge?url=...
    
그 외의 모든 동작은 `Purge`_ 와 동일하다.


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

타겟 컨텐츠의 TTL만료 시간을 현재(API호출시점)로부터 입력된 시간(초)만큼 뒤에 설정한다. 
ExpireAfter로 만료시간을 앞당겨 컨텐츠를 더 빨리 갱신하거나, 
반대로 만료시간을 늘려 원본서버 부하를 줄일 수 있다. ::

   http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

함수 호출규격은 `Purge`_ / `Expire`_ 와 유사하지만 sec파라미터(단위: 초)를 통해 
TTL만료 시간을 지정할 수 있다. 
sec가 생략된다면 기본 값은 1일(86400초)로 설정되며 0을 입력할 경우 실패한다. 
결과는 `Purge`_ / `Expire`_ 와 동일하지만 원본서버 장애여부와 상관없이 동작한다. 
결과없는 응답에 대해서는 ``<ResCodeNoCtrlTarget>`` 로 HTTP 응답코드를 설정할 수 있다.

.. note::
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
    

