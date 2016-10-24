.. admin-log:

Chapter 12. Log
******************

This chapter will explain the logs. A service both starts and ends with the logs. The logs can act as valuable assets to keep, laws to abide by, and arbitrators of system failure.

Logs are divided into global logs and virtual host logs. All logs can be configured to be on or off and share identical properties. ::

   <XXX Type="time" Unit="1440" Retention="10" Compression="OFF">ON</XXX>

-  ``Type (default: time)`` , ``Unit (default: 1440 min)`` Sets the rolling conditions for logs

   - ``time`` Rolls the log file for every configured ``unit`` of time (unit: min).
   - ``size`` Rolls the log file for every configured ``unit`` of size (unit: MB).
   - ``both`` Using a comma, time and size can be configured at the same time. For example, a configuration of ``Unit="1440, 100"`` rolls the log file every 24 hours (1440 minutes) or every 100 MB.
     
-  ``Retention (default: 10 files)`` Keeps up to the set number of log files.

-  ``Compression (default: OFF)`` Compresses the log when rolling. For example, when the file access_20140715_0000.log is rolling, it is compressed and saved as access_20140715_0000.log.gz.

If ``Type`` is set to "time" and ``Unit`` set to 10, the log will be rolled on every multiple of 10 minutes. That is, even if the service started at 2:18, the log will be rolled at 2:20, 2:30, 2:40, and so on. Likewise, if you want to roll the log every day at midnight, you can set ``Unit`` to 1440 (60 min * 20 hours). If ``Type`` is set to "time", the logs will be rolled at least once a day, and ``Unit`` cannot be set above 1440.

.. figure:: img/log_rolling1.jpg
   :align: center
   
If the log is rolled with a maximum value of 24 hours (``Unit="1440"``), the log will be recorded as seen below.

.. figure:: img/log_rolling2.jpg
   :align: center


.. toctree::
   :maxdepth: 2



.. admin-log-install:

Install Log
====================================

All details during installation/update will be recorded in the file install.log. No extra configuration is needed for this log. ::

    #DownloadURL: http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
    #DownloadTime: 13 sec
    #Target: STON 2.0.0
    #Date: 2014.03.03 16:48:35
    Prepare for STON 2.0.0 install process
        Stopping STON...
        STON stopped
    [Copying files]
        `./fuse.conf' -> `/etc/fuse.conf'
        `./libfuse.so.2' -> `/usr/local/ston/libfuse.so.2'
        `./libtbbmalloc_proxy.so' -> `/usr/local/ston/libtbbmalloc_proxy.so'
        `./start-stop-daemon' -> `/usr/sbin/start-stop-daemon'
        `./libtbbmalloc_proxy.so.2' -> `/usr/local/ston/libtbbmalloc_proxy.so.2'
        `./libtbbmalloc.so' -> `/usr/local/ston/libtbbmalloc.so'
        `./libtbbmalloc.so.2' -> `/usr/local/ston/libtbbmalloc.so.2'
        `./libtbb.so' -> `/usr/local/ston/libtbb.so'
        `./libtbb.so.2' -> `/usr/local/ston/libtbb.so.2'
        `./stond' -> `/usr/local/ston/stond'
        `./stonx' -> `/usr/local/ston/stonx'
        `./stonr' -> `/usr/local/ston/stonr'
        `./stonu' -> `/usr/local/ston/stonu'
        `./stonapi' -> `/usr/local/ston/stonapi'
        `./server.xml.default' -> `/usr/local/ston/server.xml.default'
        `./vhosts.xml.default' -> `/usr/local/ston/vhosts.xml.default'
        `./ston_format.sh' -> `/usr/local/ston/ston_format.sh'
        `./ston_diskinfo.sh' -> `/usr/local/ston/ston_diskinfo.sh'
        `./wm.sh' -> `/usr/local/ston/wm.sh'
    [Exporting config files]
        #Export so directory
        /usr/local/ston/ to ld.so.conf
        #Export sysctl to /etc/sysctl.conf
        vm.swappiness=0
        vm.min_free_kbytes=524288
        #Export sudoers for WM
        Defaults    !requiretty
        winesoft ALL=NOPASSWD: /etc/init.d/ston stop, /etc/init.d/ston start, /bin/ps -ef
    [Configuring STON daemon script]
        STON deamon activate in run-level 2345.
    [Installing sub-packages]
        curl installed.
        libjpeg installed.
        libgomp installed.
        rrdtool installed.
    [Installing WM]
        Stopping WM...
        WM stopped
        `./wm.server_default.xml' -> `/usr/local/ston/wm/tmp/conf/server_default.xml'
        `./wm.vhost_default.xml' -> `/usr/local/ston/wm/tmp/conf/vhost_default.xml'
        WM configuration found. Current WM port : 8500
        PHP module for Legacy(CentOS 5.5) installed
        `./libphp5.so.5.5' -> `/usr/local/ston/wm/modules/libphp5.so'
        WM installation almost complete. Changing WM privileges.
    Installation successfully complete



.. _admin-log-info:

Info Log
====================================

The Info log can be configured in global settings (server.xml). ::

   # server.xml - <Server><Cache>
   
   <InfoLog Type="size" Unit="1" Retention="5">ON</InfoLog>   

-  ``<InfoLog> (default: ON, Type: size, Unit: 1)`` Records operation and configuration changes in STON.
   
   
.. _admin-log-deny:

Deny Log
====================================

The Deny log can be configured in global settings (server.xml). ::

   # server.xml - <Server><Cache>

   <DenyLog Type="size" Unit="1" Retention="5">ON</DenyLog>   

-  ``<DenyLog> (default: ON, Type: size, Unit: 1)``

   Records IP addresses denied by :ref:`access-control-serviceaccess`. ::
   
      #Fields: date time c-ip deny
      2012.11.15 07:06:10 1.1.1.1 AP
      2012.11.15 07:06:26 2.2.2.2 GIN
      2012.11.15 07:06:30 3.3.3.3 3.3.3.1-255
      
   Fields are separated by spaces, and each field refers to the following:
   
   - ``date`` Date.
   - ``time`` Time.
   - ``c-ip`` Client IP.
   - ``deny`` Denial condition.
   
   
.. _admin-log-originerror:

OriginError Log
====================================

The OriginError log can be configured in global settings (server.xml). ::

   # server.xml - <Server><Cache>
   
   <OriginErrorLog Type="size" Unit="5" Retention="5" Warning="OFF">ON</OriginErrorLog>   

-  ``<OriginErrorLog> (default: OFF, Type: size, Unit: 5, Warning: OFF)``

   Records errors that occur in the origin server for all virtual hosts. Errors can be either connection timeouts and reception timeouts, and results of origin server exclusion/recovery are also logged.  ::
   
      #Fields: date time vhostname level s-domain s-ip cs-method cs-uri time-taken sc-error sc-resinfo
      2012.11.15 07:06:10 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:26 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:30 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      #2012.11.15 07:06:30 [example.com] 192.168.0.13 excluded from service
      #2012.11.15 07:06:31 [example.com] Origin server list: 192.168.0.14
      #2012.11.15 07:11:11 [example.com] 192.168.0.13 recovered back in service
      #2012.11.15 07:11:12 [example.com] Origin server list: 192.168.0.13
   
   Fields are separated by spaces, and each field refers to the following:
   
   - ``date`` Date of error.
   - ``time`` Time of error.
   - ``vhostname`` [Virtual host].
   - ``level`` [Error level (Error or Warning)].
   - ``s-domain`` Origin server domain.
   - ``s-ip`` Origin server IP.
   - ``cs-method`` HTTP Method sent by STON to the origin server.
   - ``cs-uri`` URI sent by STON to the origin server.
   - ``time-taken`` Amount of time elapsed until the system error.
   - ``sc-error`` Type of error.
   - ``sc-resinfo`` Information of server response when error occurred (separated by commas).
   
   If the ``Warning`` property is set to ``ON``, HTTP communication errors will be logged as follows.   ::
   
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 PartialResponseOnNormalRequest Res=206,Len=2635
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 ClosedWithoutResponse -
      
   HTTP communication errors can occur in the following ways.
   
   - ``ClosedWithoutResponse`` Connection closed by the origin server. HTTP response was not returned.
   - ``ClosedWhenDownloading`` Connection closed by the origin server. Desired Content-Length was not downloaded.
   - ``NotPartialResponseOnRangeRequest`` The response code to a Range request was not 206.
   - ``DifferentContentLengthOnRangeRequest`` The requested Range and Content-Length were different.
   - ``PartialResponseOnNormalRequest`` The response code to a non-Range request was 206.



.. admin-log-syslog:

SysLog Transfer
====================================

Logs can be forwarded to UDP in real time using the `syslog <http://en.wikipedia.org/wiki/Syslog>`_ protocol. All logs can be configured to be transferred via syslog. ::

   # server.xml - <Server><Cache>
   
   <InfoLog SysLog="OFF">ON</InfoLog>
   <DenyLog SysLog="OFF">ON</DenyLog>
   <OriginErrorLog SysLog="OFF">ON</OriginErrorLog>
    
-  ``SysLog``

   - ``OFF (default)`` syslog is not used.
   
   - ``ON`` Uses the ``<SysLog>`` tag configured within the current tag to transfer logs.
   
The following is an example of configuring syslog when ``<OriginErrorLog>`` is being logged. ::

   # server.xml - <Server><Cache>

   <OriginErrorLog SysLog="ON">
      <SysLog Priority="local3.info" Dest="192.168.0.1:514" />
      <SysLog Priority="user.alert" Dest="192.168.0.2" />
      <SysLog Priority="mail.debug" Dest="log.example.com" />
   </OriginErrorLog>
    
1. The ``SysLog`` property of ``<OriginErrorLog>`` is set to ``ON``.
#. The ``<SysLog>`` tag is created within ``<OriginErrorLog>``. Logs can be transferred to any number of servers.
#. The ``Priority`` property of ``<SysLog>`` is configured. This property is expressed with a combination of `Facility Levels <http://en.wikipedia.org/wiki/Syslog#Facility_levels>`_ and `Severity levels <http://en.wikipedia.org/wiki/Syslog#Severity_levels>`_.
#. The ``Dest`` property of ``<SysLog>`` is configured. This is the syslog reception server, and the reception port can be omitted if it is 514.

The syslog example recorded from the above settings can be seen below. The syslog tag is recorded as STON/{log name}. ::

    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:20 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd 1996 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:22 [ERROR] [example.com] - 192.168.0.14 GET /favicon.ico 1995 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:24 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd22 2020 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] 192.168.0.14:102 excluded from service
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] Origin server list:
    


Saving Virtual Host Logs
====================================

Logs are recorded separately for each virtual host. Even if the log is set to ``OFF``, :ref:`api-monitoring-logtrace` will work as normal. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>

   <Log Dir="/cache_log">
      ... (omitted) ...
   </Log>   

-  ``<Log>`` The ``Dir`` property configures the directory in which the logs will be recorded. The logs are saved in virtual host directories that are created under the set directory.
   


.. _admin-log-dns:

DNS Log
====================================

If the origin server is set to a Domain, the results of Resolving are recorded in the DNS log. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Dns Type="size" Unit="10" Retention="10" SysLog="OFF" Compression="OFF">ON</Dns>   
   
::

   #Fields: date time domain ttl ip-list ip-count time-taken result
   2014-07-30 12:10:33 example.com 157 173.194.127.15,173.194.127.23,173.194.127.24,173.194.127.31 4 5007 success
   2014-07-30 12:10:38 example.com 152 173.194.127.23,173.194.127.24,173.194.127.31,173.194.127.15 4 9 success
   2014-07-30 12:11:03 example.com 127 173.194.127.31,173.194.127.15,173.194.127.23,173.194.127.24 4 15007 success
   2014-07-30 12:12:53 example.com 17 173.194.127.15,173.194.127.23,173.194.127.24,173.194.127.31 4 6 success
   2014-07-30 12:23:16 test.com 0 - 0 10008 fail
   2014-07-30 12:23:21 test.com 0 - 0 5007 fail
   2014-07-30 12:23:26 test.com 0 - 0 5011 fail
   2014-07-30 12:24:38 example.com 152 173.194.127.23,173.194.127.24,173.194.127.31,173.194.127.15 4 9 success
   2014-07-30 12:25:03 example.com 127 173.194.127.31,173.194.127.15,173.194.127.23,173.194.127.24 4 15007 success

Fields are separated by spaces, and each field refers to the following:

-  ``date`` Date.
-  ``time`` Time.
-  ``domain`` Target domain.
-  ``ttl`` Time to live (Time when record is valid).
-  ``ip-list`` IP list.
-  ``ip-count`` IP count.
-  ``time-taken`` Runtime.
-  ``result`` "success" or "fail".


.. _admin-log-access:

Access Log
====================================

Records the HTTP transactions of all clients. The log is recorded when an HTTP transaction ends, whether the transfer is completed or interrupted. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <Access Type="time" Unit="1440" Retention="10" XFF="on" Form="ston" Local="Off">ON</Access>   
    
-  ``XFF``

   - ``ON (default)`` Records values of the XFF (X-Forwarded For) header sent by the client together with the client IP. If there is no header, it will be the same as ``OFF``.
   - ``OFF`` Records the client IP. 
   - ``TrimCIP`` Records the client IP if there is no XFF header, but records the XFF header without the client IP if there is a header.

-  ``Form``
   
   - ``ston (default)`` W3C standard + expansion field
   - ``apache`` Apache format
   - ``iis`` IIS format
   - ``custom`` `admin-log-access-custom`

-  ``Local``

   - ``OFF (default)`` Local communications (loopback) are not logged).
   - ``ON`` Local communications (loopback) are logged.
  
::

    #Fields: date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) sc-status sc-bytes time-taken cs-referer sc-resinfo cs-range sc-cachehit cs-acceptencoding session-id sc-content-length
    2012.06.27 16:52:24 220.134.10.5 GET /web/h.gif - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 98141 5 - Bypass+gzip+SSL3 - TCP_HIT gzip+deflate 7 1273735
    2012.06.27 16:52:26 220.134.10.5 GET /favicon.ico - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 949 2 - - - TCP_HIT gzip+deflate 35 14875
    2012.06.27 17:00:06 220.168.0.13 GET /setup.Eexe - 80 - 61.168.0.102  Mozilla/5.0+(Windows+NT+6.1;+WOW64)+AppleWebKit/536.11+(KHTML,+like+Gecko)+Chrome/20.0.1132.57+Safari/536.11 206 20971800 7008 - - 398458880-419430399 TCP_HIT - 41 89764358

Fields are separated by spaces, and each field refers to the following:

-  ``date`` Date of HTTP transaction completion.
-  ``time`` Time of HTTP transaction completion.
-  ``s-ip`` Server IP.
-  ``cs-method`` HTTP Method sent by the client.
-  ``cs-uri-stem`` URL sent by the client (excluding QueryString).
-  ``cs-uri-query`` QueryString of the URL sent by the client.
-  ``s-port`` Server port.
-  ``cs-username`` Client username.
-  ``c-ip`` Client IP. If ``XFF`` is set to ``ON``, the X-Forwarded-For header is recorded with the IP. 
-  ``cs(User-Agent)`` HTTP User-Agent sent by the client.
-  ``sc-status`` Server response code.
-  ``sc-bytes`` Bytes sent from the server (header + content).
-  ``time-taken`` Total elapsed time until an HTTP transaction is completed (ms).
-  ``cs-referer`` HTTP Referer sent by the client.
-  ``sc-resinfo`` Additional information, separated by the "+" character. If the service provides encoded content, the encoding option (gzip or deflate) is specified. For secured communications, the SSL protocol version (SSL3, TLS1, TLS1.1, TLS1.2) is specified. For bypassed communications, "Bypass" is specified.
-  ``cs-range`` Records the Range header sent by the client.
-  ``sc-cachehit`` Cache HIT results.
-  ``cs-acceptencoding`` Accept-Encoding header sent by the client.
-  ``session-id`` HTTP client sesion ID (unsigned int64).
-  ``sc-content-length`` Value of server response Content-Length header.

The access log records all HTTP transactions regardless of the success or failure of the transfer. An HTTP transaction begins when a client sends an HTTP request. If the HTTP connection is closed before STON sends a response to the client, then the transaction will still be considered completed. Both ``sc-status`` and ``sc-bytes`` will be recorded as 0. A log like this is recorded when the client closes the connection before the STON receives a response from the origin server.



.. _admin-log-access-custom:

Custom Access Log Format
====================================

The format of the access log can be customized. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <Access Form="custom">ON</Access>
   <AccessFormat>%a %A %b id=%{userid}C %f %h %H "%{user-agent}i" %m %P "%r" %s %t %T %X %I %O %R %e %S %K</AccessFormat>   
  
-  ``<Access>`` The ``Form`` property is set to ``custom``.

-  ``<AccessFormat>`` The custom log format.

With the above configuration, the access log will be recorded as follows. (#Fields are not recorded.) ::

    192.168.0.88 192.168.0.12 163276 id=winesoft; image.jpg example.com HTTP "STON" GET 80 "GET /ston/image.jpg?type=png HTTP/1.1" 200 2014-04-03 21:21:54 1 C 204 163276 1 2571978 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 63276 id=winesoft; vod.mp4 example.com HTTP "STON" POST 80 "GET /ston/vod.mp4?start=10 HTTP/1.1" 200 2014-04-03 21:21:54 12 C 304 363276 2 2571979 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 3634276 id=ston; news.html example.com HTTPS "STON" GET 443 "GET /news.html HTTP/1.1" 200 2014-04-03 21:21:54 30 X 156 2632576 1 2571980 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 6332476 id=winesoft; style.css example.com HTTP "STON" HEAD 80 "GET /style.css HTTP/1.1" 200 2014-04-03 21:21:54 10 X 234 653276 2 2571981 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 6276 id=ston; ui.js example.com HTTP "STON" GET 80 "GET /ui.js HTTP/1.1" 200 2014-04-03 21:21:54 1 X 233 63276 1 2571982 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 626 id=winesoft; hls.m4u8 example.com HTTP "STON" GET 80 "GET /hls.m4u8 HTTP/1.1" 200 2014-04-03 21:21:54 2 X 124 6312333276 2 2571983 TCP_REFRESH_HIT HTTP/1.1
  
This configuration was developed based on the `Apache log format <https://httpd.apache.org/docs/2.2/ko/mod/mod_log_config.html>`_ and there are several expansion fields. There is no restriction to the delimiters that can be used, but quotation marks ("...") should be used for items that may include spaces, such as a User-Agent.

-  ``%...a`` Client IP. ::

      192.168.0.66
      
-  ``%...A`` Server IP Address. :: 

      192.168.0.14
      
-  ``%...b`` Byte size of transfer, excluding the HTTP header. ::

      1024
      
-  ``%...{foobar}C`` The content of the "foobar" cookie in the request received by the server.  ::

      If input as %{id=}c, the cookie value corresponding to id= is recorded.
      
-  ``%...D`` Elapsed time to process the request (ms). ::

      3000
      
-  ``%...f`` File name. ::

      If /mp4/iu.mp4, iu.mp4 will be recorded.
      
-  ``%...h`` HostName. ::

      example.com
      
-  ``%...H`` Request protocol. ::

      http or https
      
-  ``%...{foobar}i`` The content of the "foobar" header in the request received by the client. ::

      If input as %{User-Agent}i, the User-Agent value is recorded.
      
-  ``%...m`` Request Method. ::

      GET or POST or HEAD
      
-  ``%...P`` Server PORT ::

      80
      
-  ``%...q`` QueryString ::

      Id=10&value=20
      
-  ``%...r`` The first line of the request (Request Line). ::

      GET /img.jpg HTTP/1.1
      
-  ``%...s`` Response code. ::

      200
      
-  ``%...t`` STON default time format.	::

      2014-01-01 15:27:02

-  ``%...{format}t`` Date shown using "format". ::

      If input as %{%Y-%m-%d %H:%M:%S}T, the output will be 2014-08-07 06:12:23.

-  ``%...T`` TimeTaken (sec). ::

      10

-  ``%...U`` ShortURI. ::

      /img/img.jpg
      
-  ``%...u`` FullURI. ::

      /img/img.jpg?session=1232&id=37

-  ``%...X`` Status when transaction is completed.
   
   - ``X`` Closed before the response is completed.
   - ``C`` Response is completed.
   
   ::
   
      C
    
-  ``%...I`` Received bytes, including the request header. ::
    
      2048
    
-  ``%...O`` Received bytes, including the response header. ::
      
      2048
      
-  ``%...R`` Response time (ms). ::

      2
      
-  ``%...e`` Session-ID. ::

      1
      
-  ``%...S`` Cache HIT results. ::

      TCP_HIT
      
-  ``%...K`` Request HTTP version.	::

      HTTP/1.1
            
-  ``%...y`` Request HTTP header size.	::

      488
            
-  ``%...z`` Response HTTP header size.	::

      362
  
If there is no value for the configured field, it will be recorded as "-". If the format is wrong, the STON default format (Form="ston") will be used instead.

The "..." in the above notations for the fields (e.g. %h %U %r %b) can be left blank, or a condition for recording can be used. If the condition is not met, "-" will be recorded. HTTP status codes can be used for conditions, and exclamation points (!) can be used for NOT conditions.


The following example only records a User-Agent when there is a 400 (Bad Request) or a 501 (Not Implemented) response. ::

    "%400,501{User-agent}i" 
  
The following example logs Referers for all abnormal responses. ::
  
    "%!200,304,302{Referer}i"



.. _admin-log-origin:

Origin Log
====================================

Records all HTTP transactions in the origin server. The log is recorded when an HTTP transaction ends, whether the transfer is completed or interrupted. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Origin Type="time" Unit="1440" Retention="10" Local="Off">ON</Origin>   
    
::

    #Fields: date time cs-sid cs-tcount c-ip cs-method s-domain cs-uri s-ip sc-status cs-range sc-sock-error sc-http-error sc-content-length cs-requestsize sc-responsesize sc-bytes time-taken time-dns time-connect time-firstbyte time-complete cs-reqinfo cs-acceptencoding sc-cachecontrol s-port sc-contentencoding session-id session-type
    2012.06.27 17:40:00 357 899 192.168.0.13 GET i.example.com /t/2.gif 115.71.9.136 200 - - - 3874 197 271 3874 20 0 0 17 3 - gzip+deflate - 80 gzip 7 cache
    2012.06.27 17:40:00 357 900 192.168.0.13 GET i.example.com /ex1.gif 115.71.9.136 200 - - - 5673 223 272 5673 24 0 0 21 3 - - - 80 - 8 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9  4 Bypass - - 80 - 7 cache
    #[ERROR:01] 2012.06.27 17:40:01 220.73.216.5 220.73.216.5 GET /web/nmb/img/main/v1/h1.gif 1824 Connect-Timeout - 11 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB1.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9 4 - max-age=3600 80 - 12 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB2.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9 4 - no-cache 80 - 35 cache
    2012.06.27 17:40:00 357 901 192.168.0.13 GET i.example.com /exB3.jpg 115.71.9.136 200 - - - 8150 189 273 8150 13 0 0 9 4 - - 80 - 35 cache

If there is an origin server failure, an error log starting with #[ERROR:xx] will be recorded. Fields are separated by spaces, and each field refers to the following:

.. figure:: img/time_taken.jpg
   :align: center
   
   Origin time measurements

-  ``date`` Date of HTTP transaction completion.
-  ``time`` Time of HTTP transaction completion.
-  ``cs-sid`` Session unique ID. HTTP transactions processed (recycled) by the same session will have the same value.
-  ``cs-tcount`` Transaction count. Records how many transactions the current session has processed. Transactions with the same ``cs-sid`` value cannot have the same ``cs-tcount`` value.
-  ``c-ip`` STON IP.
-  ``cs-method`` HTTP Method sent to the origin server.
-  ``s-domain`` Origin server domain.
-  ``cs-uri`` URI sent to the origin server.
-  ``s-ip`` Origin server IP.
-  ``sc-status`` Origin server HTTP response code.
-  ``cs-range`` Value of Range request sent to the origin server.
-  ``sc-sock-error`` Socket error code (1=Transfer timeout, 2=Transfer delay, 3=Connection close).
-  ``sc-http-error`` Log of response code when origin server returns either 4xx or 5xx responses.
-  ``sc-content-length`` Content Length sent by the origin server.
-  ``cs-requestsize (unit: bytes)`` Size of the HTTP request header sent to the origin server.
-  ``sc-responsesize (unit: bytes)`` Size of the HTTP header of the origin server's response.
-  ``sc-bytes (unit: bytes)`` Received content size (header excluded).
-  ``time-taken (unit: ms)`` Total elapsed time until the HTTP transaction is completed. If the session is not recycled, the socket connection time is included.
-  ``time-dns (unit: ms)`` Elapsed time during DNS query.
-  ``time-connect (unit: ms)`` Elapsed time until a socket is established with the origin server.
-  ``time-firstbyte (unit: ms)`` Elapsed time from a sent request to a received response.
-  ``time-complete (unit: ms)`` Elapsed time from the first response to completion.
-  ``cs-reqinfo`` Additional information. Separated by the "+" character. Recorded as "Bypass" for bypass communications and "PrivateBypass" for private bypass communications.
-  ``cs-acceptencoding`` Recorded as "gzip+deflate" if compressed content is requested from the origin server.
-  ``sc-cachecontrol`` Cache-control header sent by the origin server.
-  ``s-port`` Origin server port.
-  ``sc-contentencoding`` Content-Encoding header sent by the origin server.
-  ``session-id`` HTTP client session ID that created the origin server request (unsigned int64).
-  ``session-type`` Session type requested from the origin server.

   -  ``cache`` Sessions used for caching.
   -  ``recovery`` Sessions used for recovery in :ref:`origin_exclusion_and_recovery`.
   -  ``healthcheck`` Sessions used by :ref:`origin-health-checker`.


.. admin-log-monitoring:

Monitoring Log
====================================

Records the average statistics of the last five minutes. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Monitoring Type="size" Unit="10" Retention="10" Form="json">ON</Monitoring>
  
-  ``Form`` Assigns the log format. (``json`` or ``xml``)



.. admin-log-filesystem:

FileSystem Log
====================================

Records all File I/O transactions generated using the :ref:`filesystem`. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <FileSystem Type="time" Unit="1440" Retention="10">ON</FileSystem>
  
Logging occurs when the File I/O transaction is completed. The time of completion differs based on the type of cs-method. ::

    #Fields: date time cs-method cs-path sc-status sc-bytes response-time time-taken sc-cachehit attr session-id
    2012.06.27 16:52:24 ATTR /t 200 0 100 100 TCP_HIT FOLDER 1
    2012.06.27 16:52:24 ATTR /t/2.gif 200 0 100 100 TCP_HIT FILE 1
    2012.06.27 16:52:24 OPEN /file.txt 200 0 100 2000 TCP_HIT FILE 2
    2012.06.27 16:52:24 READ /file.txt 200 1024768 100 2000 TCP_HIT FILE 2
    
-  ``date`` Date of File I/O transaction completion.
-  ``time`` Time of File I/O transaction completion.
-  ``cs-method`` File I/O access type. One of the following three can be used:

   -  ``ATTR`` getattr function call. Logs when the function is returned.
   -  ``OPEN`` File is opened but not READ. Logs when the file is closed.
   -  ``READ`` File is opened and READ. Logs when the file is closed.
   
-  ``cs-path`` Access path.
-  ``sc-status`` Response code. The following show the failure codes, as well as the code for normal service (200).

   -  ``200`` Normal service.
   -  ``301`` Bypass required.
   -  ``302`` Service denied.
   -  ``303`` Redirect required.
   -  ``400`` Invalid request.
   -  ``401`` Unable to find the virtual host.
   -  ``402`` Initialization failure from the origin server.
   -  ``500`` Object initialization failure.
   -  ``501`` Object open failure.
   -  ``502`` Save path generation failure.
   -  ``503`` Memory initialization failure.
   -  ``504`` Emergency status.
   -  ``600`` Timeout during file service standby.
   -  ``601`` Timeout during file data service standby.
   -  ``602`` File initialization failure during file service standby.
   -  ``603`` Data initialization failure during file data service standby.
   -  ``701`` Invalid offset.
   -  ``702`` Specific file section load failure.
   -  ``703`` Not enough memory.
   -  ``704`` Origin session generation failure.
   
-  ``sc-bytes`` Read byte size.
-  ``response-time`` Elapsed time from the function call to the connection to the service object.
-  ``time-taken`` Elapsed time from the function call to the completion of the File I/O transaction.
-  ``sc-cachehit`` Cache HIT results.
-  ``attr`` FILE or FOLDER.
-  ``session-id`` File I/O session ID (unsigned int64).

   .. note::
   
      ``session-id`` is assigned when the Client (HTTP or File I/O) Context is generated. In the general file process flow of Open -> Read -> Close, the Client Context is constructed at Open and destructed at Close. On the other hand the getattr function is an "atomic function", so the Client Context is created/destructed every time, and a new ``session-id`` is always assigned.

  
  
.. _admin-log-ftp:

FTP Transfer
====================================

When the log is rolled, it is also uploaded using the designated FTP client.


.. _admin-log-ftpclient:

FTP Client
---------------------

Configures the FTP client. Rolled logs are uploaded to the FTP server in real time.

.. figure:: img/conf_ftpclient.png
   :align: center
   
   FTP client structure and operation.

FTP clients exist outside of STON, as seen in the image above. STON is responsible for inputting logs stored locally into the FTP client queue, and has no effect on FTP operation. The FTP client can then process the uploads based on its own configuration.

The FTP clients can be configured in global settings (server.xml). ::

   # server.xml - <Server>

   <Ftp Name="backup1">
      <Mode>Passive</Mode>
      <Address>ftp.winesoft.co.kr:21</Address>
      <Account>
         <ID>test</ID>
         <Password>12345abc</Password>
      </Account>
      <ConnectTimeout>10</ConnectTimeout>
      <TransferTimeout>600</TransferTimeout>
      <TrafficCap>0</TrafficCap>
      <DeleteUploaded>OFF</DeleteUploaded>
      <BackupOnFail>OFF</BackupOnFail>
      <UploadPath>/log_backup/%v/%s-%e.%p.log</UploadPath>
      <Transfer Time="Rotate" />
   </Ftp>
        
   <Ftp Name="backup2">
      <Mode>Active</Mode>
      <Address>192.168.0.14:21</Address>
      <Account>
         <ID>test</ID>
         <Password>qwerty</Password>
      </Account>
      <ConnectTimeout>3</ConnectTimeout>
      <TransferTimeout>100</TransferTimeout>
      <TrafficCap>10240</TrafficCap>
      <DeleteUploaded>ON</DeleteUploaded>
      <BackupOnFail>ON</BackupOnFail>            
      <Transfer Time="Static">04:00</Transfer>
   </Ftp>   

-  ``<Ftp>`` Configures FTP clients. Individual names can be configured with the ``Name`` property.

   - ``Mode (default: Passive)`` Connection mode (``Passive`` or ``Active``).
   - ``Address`` FTP address.
   - ``Account`` FTP account. To encrypt the password (e.g. qwerty), the API below can be used. ::
     
        /command/encryptpassword?plain=qwerty
        
     The encrypted password can then be configured as follows. ::
     
        <Password Type="enc">dXR9k0xNUZVVYQsK5Bi1cg==</Password>
     
   - ``ConnectTimeout`` Connection pending time.
   - ``TransferTimeout`` Transfer pending time.
   - ``TrafficCap (unit: KB)`` If set to greater than 0, the maximum transfer bandwidth is configured.
   - ``DeleteUploaded (default: OFF)`` Deletes the corresponding log after transfer completion.
   - ``BackupOnFail (default: OFF)`` Backs up the corresponding log in the following path so that the log is not deleted on a transfer failure. ::
     
        /usr/local/ston/stonb/backup/
     
     Backup logs are not retransmitted and will not be deleted unless done deliberately by the administrator.
   
   - ``UploadPath`` Configures the upload path.
     If not configured, the path becomes "/virtual host/". For example, logs for example.com are uploaded to the /example.com/ directory.
     
     - ``%{time format}s`` Log start time.
     - ``%{time format}e`` Log end time.
     - ``%p`` Prefix.
     - ``%v`` Virtual host name.
     - ``%h`` Device HOST name.
     
     For example, if the following configuration is used, ::
     
        # server.xml - <Server><Ftp>
        
        <UploadPath>/log_backup/%v/%s-%e.%p.log</UploadPath>
     
     the upload path will be as follows. ::
     
        /log_backup/example.com/200140722_0000-200140722_2300.access.log
        
   - ``Transfer`` Determines the log transfer time. The ``Type`` property determines the format of the value.
   
     - ``Rotate (default)`` Transfers the log immediately after rolling. Does not require a value.
     - ``Static`` Transfers the log once a day at a specific time. For example, if set to 04:00, a transfer will occur every day at 4 am.
     - ``Interval`` Transfers the log after every set interval of time. For example, if set to 4, a transfer will occur every four hours.
     
     It is important to configure a proper logging policy to ensure that rolling does not occur while logs are being transferred.
     

FTP clients use curl.


.. admin-log-ftplog:

FTP Log
---------------------

FTP logs are merged and saved in /usr/local/ston/sys/stonb/stonb.log. ::

    #Fields: date time local-path cs-url file-size time-taken sc-status sc-error-msg
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10006 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/access_20140423_1700.log ftp://192.168.0.14:21/winesoft.co.kr/access_20140423_1700.log 260 60 success "-"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10008 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/filesystem_20140423_080000.log ftp://192.168.0.14:21/winesoft.co.kr/filesystem_20140423_080000.log 179 60 success "-"

Fields are separated by spaces, and each field refers to the following:

-  ``date`` Date.
-  ``time`` Time.
-  ``local-path`` Local path of the log to be transferred.
-  ``cs-url`` FTP address to transfer the log to.
-  ``file-size`` Transfer file size.
-  ``time-taken (unit: ms)`` Elapsed time of transfer.
-  ``sc-status`` Transfer success/failure.
-  ``sc-error-msg`` Curl error message on transfer failure.



.. admin-log-ftptransfer:

Log FTP Transfer
---------------------

When the log is rolling, the designated `FTP client` will be used to upload the log. If separated by commas, multiple `FTP clients` can be used at the same time. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>
   
   <Log>
      <Access Ftp="backup1, backup2">ON</Access>
      <Origin Ftp="backup_org">ON</Origin>
      <Monitoring Ftp="backup1">ON</Monitoring>
      <FileSystem Ftp="backup2">ON</FileSystem>   
   </Log>
    
-  ``Ftp`` The `FTP client` to be used.

The log is uploaded to ftp://{FTP server address}/{virtual host name}/{rolled log name}. For example, the upload address of the rolled log "access_20140424_0000.log" of the virtual host "example.com" in the server "ftp.dummy.com" will be `ftp://ftp.dummy.com/example.com/access_20140424_0000.log`.