.. admin-log:

Chapter 11. Log
******************

This chapter will explain the log.
A service begins with the log and ends with the log.
The log is the most valuable asset to keep, and a legislation to obey, and an arbitrator of system failure.

There are global and virtual host logs. 
All logs can be turned on or off, and they have identical properties. ::

   <XXX Type="time" Unit="1440" Retention="10" Compression="OFF">ON</XXX>

-  ``Type (default: time)`` and ``Unit (default: 1440 minutes)`` set log rolling conditions.

   - ``time`` Rolls the log file for every configured ``unit`` of time (unit: minute).
   - ``size`` Rolls the log file for every configured ``unit`` of size (unit: MB).
   - ``both`` By using a comma(,), time and size can be configured at the same time.
     For example, the Unit="1440, 100" configuration rolls the log file every 24 hours (1,440 minutes) or every 100MB.
     
-  ``Retention (default: 10 files)`` Keeps the set number of log files.

-  ``Compression (default: OFF)`` Compresses the log when rolling.
   For example, when the access_20140715_0000.log file is rolling, it is compressed and saved as access_20140715_0000.log.gz.

If the ``Type`` is set to "time" and the ``Unit`` is set to 10, the log is rolled for every multiple of 10 minutes.
For example, even if the service started at 2:18, logs will be rolled at 2:20, 2:30, 2:40, and so on. 
Likewise, if you want to roll the log once at midnight, you can set the ``Unit`` to 1,440 (60 minutes X 24 hours).
In the ``time`` configuration, the log will be rolled at least once a day; therefore, the maximum value of ``Unit`` cannot exceed 1,440.

.. figure:: img/log_rolling1.jpg
   :align: center
   
If you set the maximum value of 1,440 for "Unit", the log will be recorded as shown in the figure below.

.. figure:: img/log_rolling2.jpg
   :align: center


.. toctree::
   :maxdepth: 2



.. admin-log-install:

Install Log
====================================

All details during installation/update will be recorded in the install.log.
No extra configuration is needed for this log. ::

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

The Info log can be configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>
   
   <InfoLog Type="size" Unit="1" Retention="5">ON</InfoLog>   

-  ``<InfoLog> (default: ON, Type: size, Unit: 1)``   
   Records operation and configuration changes in STON.
   
   
.. _admin-log-deny:

Deny Log
====================================

The Deny log is configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>

   <DenyLog Type="size" Unit="1" Retention="5">ON</DenyLog>   

-  ``<DenyLog> (default: ON, Type: size, Unit: 1)``

   The deny log records blocked IP addresses by :ref:`access-control-serviceaccess`. ::
   
      #Fields: date time c-ip deny
      2012.11.15 07:06:10 1.1.1.1 AP
      2012.11.15 07:06:26 2.2.2.2 GIN
      2012.11.15 07:06:30 3.3.3.3 3.3.3.1-255
      
   Each field is distinguished by an empty space, and each specifies the following:
   
   - ``date`` Date
   - ``time`` Time
   - ``c-ip`` Client's IP
   - ``deny`` Denied condition
   
   
.. _admin-log-originerror:

OriginError Log
====================================

The OriginError log is configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>
   
   <OriginErrorLog Type="size" Unit="5" Retention="5" Warning="OFF">ON</OriginErrorLog>   

-  ``<OriginErrorLog> (default: OFF, Type: size, Unit: 5, Warning: OFF)``

   The OriginError log records only the errors that occur in the origin server of all virtual hosts. 
   Errors consist of connection timeouts and receive timeouts, and exclusion/recovery results of the origin server are logged. ::
   
      #Fields: date time vhostname level s-domain s-ip cs-method cs-uri time-taken sc-error sc-resinfo
      2012.11.15 07:06:10 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:26 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:30 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      #2012.11.15 07:06:30 [example.com] 192.168.0.13 excluded from service
      #2012.11.15 07:06:31 [example.com] Origin server list: 192.168.0.14
      #2012.11.15 07:11:11 [example.com] 192.168.0.13 recovered back in service
      #2012.11.15 07:11:12 [example.com] Origin server list: 192.168.0.13
   
   Each field is distinguished by an empty space, and field specifies the following:
   
   - ``date`` Date of error occurrence
   - ``time`` Time of error occurrence
   - ``vhostname`` [Virtual host]
   - ``level`` [Error level(Error or Warning)]
   - ``s-domain`` Origin server domain
   - ``s-ip`` Origin server IP
   - ``cs-method`` HTTP Method sent to the origin server from STON
   - ``cs-uri`` URI sent to the origin server from STON
   - ``time-taken`` Amount of time that elapsed until the system error
   - ``sc-error`` Type of error
   - ``sc-resinfo`` Information of response from the server when error occurred (distinguished with a comma)
   
   If the ``Warning`` property is set to ``ON``, the following erroneous HTTP communication will be logged. ::
   
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 PartialResponseOnNormalRequest Res=206,Len=2635
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 ClosedWithoutResponse -
      
   An erroneous HTTP communication occurs in the following cases:
   
   - ``ClosedWithoutResponse`` Connection closed by the origin server. HTTP response is not returned.
   - ``ClosedWhenDownloading`` Connection closed by the origin server. Desired Content-Length is not downloaded.
   - ``NotPartialResponseOnRangeRequest`` Range request is not returned by 206 response code.
   - ``DifferentContentLengthOnRangeRequest`` Content-Length is not matching with the requested Range.
   - ``PartialResponseOnNormalRequest`` Response code 206 is returned for a non-Range requests.



.. admin-log-syslog:

SysLog Transfer
====================================

Use the `syslog <http://en.wikipedia.org/wiki/Syslog>`_ protocol to forward logs with UDP in real time. 
All logs can be configured to be transferred to syslog. ::

   # server.xml - <Server><Cache>
   
   <InfoLog SysLog="OFF">ON</InfoLog>
   <DenyLog SysLog="OFF">ON</DenyLog>
   <OriginErrorLog SysLog="OFF">ON</OriginErrorLog>
    
-  ``SysLog``

   - ``OFF (default)`` Does not use syslog.
   
   - ``ON`` Transfer the log to ``<SysLog>`` that is configured underneath the current tag.
   
The following is an example of configuring syslog when ``<OriginErrorLog>`` is being logged. ::

   # server.xml - <Server><Cache>

   <OriginErrorLog SysLog="ON">
      <SysLog Priority="local3.info" Dest="192.168.0.1:514" />
      <SysLog Priority="user.alert" Dest="192.168.0.2" />
      <SysLog Priority="mail.debug" Dest="log.example.com" />
   </OriginErrorLog>
    
1. Set ``Syslog`` properties in the ``<OriginErrorLog>``.
#. Create the ``<SysLog>`` tag underneath the ``<OriginErrorLog>``. The log can be transferred to a configured number of servers at the same time.
#. Configure the ``Priority`` properties in the ``<SysLog>``. 
   The expression consists of a combination of `Facility Levels <http://en.wikipedia.org/wiki/Syslog#Facility_levels>`_ in syslog and 
   `Severity levels <http://en.wikipedia.org/wiki/Syslog#Severity_levels>`_.
#. Configure the ``Dest`` properties in the ``<SysLog>``. ``Dest`` stands for the syslog reception server. If the reception port is 514, it can be omitted.

The configuration below is a sys log example based on the above configurations. 
The tag in the syslog is logged as STON/{log name}. ::

    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:20 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd 1996 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:22 [ERROR] [example.com] - 192.168.0.14 GET /favicon.ico 1995 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:24 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd22 2020 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] 192.168.0.14:102 excluded from service
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] Origin server list:
    


Saving Virtual Host Logs
====================================

Each virtual host's logs are recorded separately. 
Even if the log is set to ``OFF``, :ref:`api-monitoring-logtrace` works as normal. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>

   <Log Dir="/cache_log">
      ... (skip) ...
   </Log>   

-  ``<Log>`` Configures the directory with the ``Dir`` attribute where the log will be recorded. 
   The log is created in the virtual host directory that is underneath the configured directory.
   


.. _admin-log-dns:

DNS Log
====================================

If the origin server is set as a Domain, the DNS log records the Resolving result. ::

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

Each field is distinguished by an empty space, and each specifies the following:

-  ``date`` Date
-  ``time`` Time
-  ``domain`` Target domain
-  ``ttl`` (Time To Live) Valid time of the record
-  ``ip-list`` IP list
-  ``ip-count`` Number of IP
-  ``time-taken`` Running time
-  ``result`` Success or failure


.. _admin-log-access:

Access Log
====================================

Records HTTP transactions of all clients. 
This log is recorded when an HTTP transaction--either a transfer completion or transfer interruption--is completed. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <Access Type="time" Unit="1440" Retention="10" XFF="on" Form="ston" Local="Off">ON</Access>   
    
-  ``XFF``

   - ``ON (default) `` Logs X-Fowarded-For header values sent from clients. Without the header, this is the same as ``OFF``.
   - ``OFF `` Records client IPs.
   - ``TrimCIP``Logs client IPs if XFF header is available. Otherwise, logs XFF header only (except client IPs).


-  ``Form``
   
   - ``ston (default)`` W3C standard + expansion field
   - ``apache`` Apache format
   - ``iis`` IIS format
   - ``custom`` `admin-log-access-custom`

-  ``Local``

   - ``OFF (default)`` Local communications (Loopback) are not logged.
   - ``ON`` Local communications (Loopback) are logged.
  
::

    #Fields: date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) sc-status sc-bytes time-taken cs-referer sc-resinfo cs-range sc-cachehit cs-acceptencoding session-id sc-content-length
    2012.06.27 16:52:24 220.134.10.5 GET /web/h.gif - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 98141 5 - Bypass+gzip+SSL3 - TCP_HIT gzip+deflate 7 1273735
    2012.06.27 16:52:26 220.134.10.5 GET /favicon.ico - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 949 2 - - - TCP_HIT gzip+deflate 35 14875
    2012.06.27 17:00:06 220.168.0.13 GET /setup.Eexe - 80 - 61.168.0.102  Mozilla/5.0+(Windows+NT+6.1;+WOW64)+AppleWebKit/536.11+(KHTML,+like+Gecko)+Chrome/20.0.1132.57+Safari/536.11 206 20971800 7008 - - 398458880-419430399 TCP_HIT - 41 89764358

Each field is distinguished by an empty space, and each field specifies the following:

-  ``date`` Completed date of HTTP transaction
-  ``time`` Completed time of HTTP transaction
-  ``s-ip`` Server IP
-  ``cs-method`` HTTP Method sent from the client
-  ``cs-uri-stem`` A URL (excluding the QueryString) sent from the client
-  ``cs-uri-query`` QueryString of the URL sent from the client
-  ``s-port`` Server port
-  ``cs-username`` Client username
-  ``c-ip`` If the XFF configuration of the client IP is set to "ON", records X-Forwarded-For header value
-  ``cs(User-Agent)`` HTTP User-Agent sent from the client
-  ``sc-status`` Server response code
-  ``sc-bytes`` Bytes sent from the server (header + contents)
-  ``time-taken`` Total elapsed time until an HTTP transaction is completed (in milliseconds)
-  ``cs-referer`` HTTP Referer sent from the client
-  ``sc-resinfo`` Additional information, distinguished by the "+" character. 
   If encoded content is serviced, the encoding option(gzip or defalte) is specified. 
   Also the security method (SSL3 or TLS1) is specified for a secured communications. 
   "Bypass" will be specified for a bypassed communication.
   
-  ``cs-range`` Logs range header sent from the client
-  ``sc-cachehit`` Cache HIT results
-  ``cs-acceptencoding`` Accept-Encoding header sent from the client
-  ``session-id`` HTTP client session ID (unsigned int64)
-  ``sc-content-length`` Server response Content-Length header value

The access log records all HTTP transactions regardless of the success or failure of the transfer. 
An HTTP transaction starts when a client sends an HTTP request. 
If the HTTP connection is closed before STON sends a response to the client, the corresponding HTTP transaction is still considered complete. 
Both the ``sc-status`` and ``sc-bytes`` of the log will be recorded as 0. 
This kind of log is recorded when the client closes the connection before STON receives a response from the origin server.



.. _admin-log-access-custom:

Custom Access Log Format
====================================

The following configurations enables the use of the customized access log format. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <Access Form="custom">ON</Access>
   <AccessFormat>%a %A %b id=%{userid}C %f %h %H "%{user-agent}i" %m %P "%r" %s %t %T %X %I %O %R %e %S %K</AccessFormat>   
  
-  The ``Form`` attribute of ``<Access>`` is set to ``custom``.

-  ``<AccessFormat>`` Custom log format.

The following Access log will be recorded for the above configurations (#Fields are not recorded). ::

    192.168.0.88 192.168.0.12 163276 id=winesoft; image.jpg example.com HTTP "STON" GET 80 "GET /ston/image.jpg?type=png HTTP/1.1" 200 2014-04-03 21:21:54 1 C 204 163276 1 2571978 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 63276 id=winesoft; vod.mp4 example.com HTTP "STON" POST 80 "GET /ston/vod.mp4?start=10 HTTP/1.1" 200 2014-04-03 21:21:54 12 C 304 363276 2 2571979 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 3634276 id=ston; news.html example.com HTTPS "STON" GET 443 "GET /news.html HTTP/1.1" 200 2014-04-03 21:21:54 30 X 156 2632576 1 2571980 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 6332476 id=winesoft; style.css example.com HTTP "STON" HEAD 80 "GET /style.css HTTP/1.1" 200 2014-04-03 21:21:54 10 X 234 653276 2 2571981 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 6276 id=ston; ui.js example.com HTTP "STON" GET 80 "GET /ui.js HTTP/1.1" 200 2014-04-03 21:21:54 1 X 233 63276 1 2571982 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 626 id=winesoft; hls.m4u8 example.com HTTP "STON" GET 80 "GET /hls.m4u8 HTTP/1.1" 200 2014-04-03 21:21:54 2 X 124 6312333276 2 2571983 TCP_REFRESH_HIT HTTP/1.1
  
This configuration is developed based on `Apache log format <https://httpd.apache.org/docs/2.2/ko/mod/mod_log_config.html>`_ and there are several expansion fields. 
Specifiers in each field do not have any restrictions, but when you are using a space as a specifier,
you should use double quotation marks for items that could include space, such as a User-Agent.

-  ``%...a`` Client IP ::

      192.168.0.66
      
-  ``%...A`` Server IP address :: 

      192.168.0.14
      
-  ``%...b`` Size of transferred bytes, excluding the HTTP header ::

      1024
      
-  ``%...{foobar}C`` The Foobar cookie content in the request that the server received  ::

      If %{id=}c format is used, then the id value of the Cookie will be logged.
      
-  ``%...D`` Elapsed time to process the request (in milliseconds) ::

      3000
      
-  ``%...f`` File name ::

      If the file name is /mp4/iu.mp4, log iu.mp4
      
-  ``%...h`` HostName ::

      example.com
      
-  ``%...H`` Request protocol ::

      http or https
      
-  ``%...{foobar}i`` Foobar content: header of the request that the server received ::

      If %{User-Agent}i format is used, then the User-Agent value will be logged
      
-  ``%...m`` Request Method ::

      GET or POST or HEAD
      
-  ``%...P`` Server PORT ::

      80
      
-  ``%...q`` QueryString ::

      Id=10&value=20
      
-  ``%...r`` The first line of the request(Request Line) ::

      GET /img.jpg HTTP/1.1
      
-  ``%...s`` Response code ::

      200
      
-  ``%...t`` STON default time format	::

      2014-01-01 15:27:02

-  ``%...{format}t`` Date format defined in Format ::

      If %{%Y-%m-%d %H:%M:%S}T format is used, the date will be recorded as 2014-08-07 06:12:23 in the log.

-  ``%...T`` TimeTaken (in seconds) ::

      10

-  ``%...U`` ShortURI ::

      /img/img.jpg

-  ``%...u`` FullURI ::

      /img/img.jpg?session=1232&id=37

-  ``%...X`` Status when the transaction is completed
   
   - ``X`` Terminated before the response is completed
   - ``C`` Response is completed
   
   ::
   
      C
    
-  ``%...I`` Received bytes, including the request header ::
    
      2048
    
-  ``%...O`` Transmitted bytes, including the response header ::
      
      2048
      
-  ``%...R`` Response time (in milliseconds) ::

      2
      
-  ``%...e`` Session-ID ::

      1
      
-  ``%...S`` Caching HIT result ::

      TCP_HIT
      
-  ``%...K`` Request HTTP version	::

      HTTP/1.1
  
"-" indicates that the configured field value does not exist. 
If the format is wrong, the STON default format (Form="ston") will be adopted.
  
As shown in the above table, you can leave empty spaces ("...") in each field (e.g. "%h %U %r %b) as blanks or specify a record condition (if not satisfied, "-" is logged). 
The condition can be configured with an HTTP status code list or a NOT condition can be set with an exclamation mark (!). 

The following example only logs the User-agent when there is a 400 (Bad Request) error or a 501 (Not Implemented) error. ::

    "%400,501{User-agent}i" 
  
The following example logs Referers of all abnormal requests. ::
  
    "%!200,304,302{Referer}i"



.. _admin-log-origin:

Origin Log
====================================

This logs all HTTP transactions in the origin server. 
A log is recorded when an HTTP transaction, such as a transfer completion or transfer interruption, is completed. ::

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

If there is an origin server failure, an error log starting with #[ERROR:xx] will be recorded. 
Each field is distinguished by an empty space, and each field specifies the following:

.. figure:: img/time_taken.jpg
   :align: center
   
   Time measurement section of the origin

-  ``date`` HTTP transaction completed date
-  ``time`` HTTP transaction completed time
-  ``cs-sid`` Unique ID of the session. HTTP transactions that are processed (recycled) with the same session will have the same value.
-  ``cs-tcount`` Transaction count. This value counts how many HTTP transactions are processed during a session. Transactions with an identical ``cs-sid`` cannot have the same value.
-  ``c-ip`` IP of STON
-  ``cs-method`` HTTP Method sent to the origin server
-  ``s-domain`` Origin server domain
-  ``cs-uri`` URI sent to the origin server
-  ``s-ip`` Origin server IP
-  ``sc-status`` Origin server HTTP response code
-  ``cs-range`` Range request value sent to the origin server
-  ``sc-sock-error`` Socket error code (1=Transfer timeout, 2=Transfer delay, 3=Connection close)
-  ``sc-http-error`` Logs the response code when the origin server returns either 4xx or 5xx responses
-  ``sc-content-length`` Content Length sent by the origin server
-  ``cs-requestsize (unit: Bytes)`` The size of the HTTP request header sent to the origin server
-  ``sc-responsesize (unit: Bytes)`` The size of HTTP header that the origin server replied
-  ``sc-bytes (unit: Bytes)`` Received content size (header excluded)
-  ``time-taken (unit: ms)`` Total elapsed time before the HTTP transaction is completed. If the session is not recycled, the socket connection time will be added up
-  ``time-dns (unit: ms)`` Time spent on DNS query
-  ``time-connect (unit: ms)`` Time spent for establishing a socket with the origin server
-  ``time-firstbyte (unit: ms)`` Elapsed time from request transmission to response reception
-  ``time-complete (unit: ms)`` Elapsed time from the first response to completion
-  ``cs-reqinfo`` Additional information, distinguished by the "+" character.  "Bypass" will be logged for a bypass communication, and "PrivateBypass" will be logged for a private bypass communication.
-  ``cs-acceptencoding`` If compressed content is requested to the origin server, "gzip+deflate" will be logged.
-  ``sc-cachecontrol`` Cache-control header sent from the origin server
-  ``s-port`` Origin server port
-  ``sc-contentencoding`` Content-Encoding header sent from the origin server
-  ``session-id`` The HTTP client session ID(unsigned int64) that caused the origin server request
-  ``session-type`` Session type requested to the origin server

   -  ``cache`` Sessions that are used for caching
   -  ``recovery`` Sessions that are used for recovery in the :ref:`origin_exclusion_and_recovery`
   -  ``healthcheck`` Sessions that are used by :ref:`origin-health-checker`


.. admin-log-monitoring:

Monitoring Log
====================================

Logs average stats of the last five minutes. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Monitoring Type="size" Unit="10" Retention="10" Form="json">ON</Monitoring>
  
-  ``Form`` Specifies the log format. ( ``json`` or ``xml`` )



.. admin-log-filesystem:

FileSystem Log
====================================

Logs all File I/O transactions generated by the :ref:`filesystem`. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <FileSystem Type="time" Unit="1440" Retention="10">ON</FileSystem>
  
FileSystem is logged when the File I/O transaction is completed. 
The transaction completion point differs based upon the type of cs-method. ::

    #Fields: date time cs-method cs-path sc-status sc-bytes response-time time-taken sc-cachehit attr session-id
    2012.06.27 16:52:24 ATTR /t 200 0 100 100 TCP_HIT FOLDER 1
    2012.06.27 16:52:24 ATTR /t/2.gif 200 0 100 100 TCP_HIT FILE 1
    2012.06.27 16:52:24 OPEN /file.txt 200 0 100 2000 TCP_HIT FILE 2
    2012.06.27 16:52:24 READ /file.txt 200 1024768 100 2000 TCP_HIT FILE 2
    
-  ``date`` File I/O transaction completion date
-  ``time`` File I/O transaction completion time
-  ``cs-method`` File I/O access type. One of the following can be used:

   -  ``ATTR`` getattr function call. Logs when the function is returned
   -  ``OPEN`` File opened without READ. Logs when the file is closed
   -  ``READ`` File opened with READ. Logs when the file is closed
   
-  ``cs-path`` Access path
-  ``sc-status`` Response code. The following, except the normal service (200), are failure codes:

   -  ``200`` Normal service
   -  ``301`` Bypass required
   -  ``302`` Service denied
   -  ``303`` Redirect required
   -  ``400`` Invalid request
   -  ``401`` Unable to find the virtual host
   -  ``402`` Initialization failure from the origin server
   -  ``500`` Object initialization failure
   -  ``501`` Object open failure
   -  ``502`` Generating save path failure
   -  ``503`` Memory initialization failure
   -  ``504`` Emergency status
   -  ``600`` Timeout during file service standby
   -  ``601`` Timeout during file data service standby
   -  ``602`` File initialization failure during file service standby
   -  ``603`` Data initialization failure during file data service standby
   -  ``701`` Invalid offset
   -  ``702`` Specific file section load failure
   -  ``703`` Not enough memory
   -  ``704`` Generating origin session failure
   
-  ``sc-bytes`` Read byte size
-  ``response-time`` Time elapsed between the function call and connection to the service object
-  ``time-taken`` Time elapsed between the function call and the completion to the File I/O Transaction.
-  ``sc-cachehit`` Cache HIT result.
-  ``attr`` FILE or FOLDER
-  ``session-id`` File I/O session ID (unsigned int64)

   .. note::
   
      ``session-id`` is allocated when the Client(HTTP or File I/O) Context is generated. 
      In the general process of Open -> Read -> Close, the Client Context is created at Open and destructed at Close. 
      On the other hand, the getattr function is an "atomic function", wherein the Client Context is constantly created/destructed, so it is always assigned with a new session-id.

  
  
.. _admin-log-ftp:

FTP Transfer
====================================

when the log is rolling, it is uploaded through the designated FTP client. 


.. _admin-log-ftpclient:

FTP Client
---------------------

Configures the FTP client. 
The rolled log is uploaded to the FTP server in real time.

.. figure:: img/conf_ftpclient.png
   :align: center
   
   FTP client architecture and operation

An FTP client exists outside STON, as seen above. 
STON inserts local logs into the FTP client queue without administrating FTP operation. 
Then the FTP client processes the upload based on its configuration.

FTP clients can be configured in the global setting (server.xml). ::

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

-  ``<Ftp>`` Configures FTP clients. A unique name for each can be configured with the ``Name`` attribute.

   - ``Mode (default: Passive)`` Connection mode ( ``Passive`` or ``Active`` )
   - ``Address`` FTP address. 
   - ``Account`` FTP account. If you want to encrypt the password (e.g. qwerty), the API below can be adopted. ::
     
        /command/encryptpassword?plain=qwerty
        
     The encrypted password can be configured as follows: ::
     
        <Password Type="enc">dXR9k0xNUZVVYQsK5Bi1cg==</Password>
     
   - ``ConnectTimeout`` Connection pending time
   - ``TransferTimeout`` Transfer pending time
   - ``TrafficCap (unit: KB)`` If this value is greater than 0, it configures the maximum transfer bandwidth.
   - ``DeleteUploaded (default: OFF)`` Deletes the corresponding log after transfer completion.
   - ``BackupOnFail (default: OFF)`` Backs up corresponding log so that the log remains alive after a transfer failure. ::
     
        /usr/local/ston/stonb/backup/
     
     Backup logs are not retransmitted and will not be deleted until the administrator deletes them.
   
   - ``UploadPath`` Configures the upload path.
     
     - ``%{time format}s`` Log start time
     - ``%{time format}e`` Log end time
     - ``%p`` Prefix
     - ``%v`` Virtual host name
     - ``%h`` Device HOST name
     
     If the configuration below is used, ::
     
        # server.xml - <Server><Ftp>
        
        <UploadPath>/log_backup/%v/%s-%e.%p.log</UploadPath>
     
     the upload path will be as follows: ::
     
        /log_backup/example.com/200140722_0000-200140722_2300.access.log
        
   - ``Transfer`` Determines the log transfer time. The ``Type`` attribute will determine the value format.
   
     - ``Rotate (default)`` Transfers the log right after rolling and does not have a value.
     - ``Static`` Transfers the log once a day at a specific time. For example, if this value is set to 04:00, a transfer will occur at 4a.m..
     - ``Interval`` Transfers the log every set interval of time. For example, if this value is set to 4, a transfer will occur every four hours.
     
     If you want to configure the transfer time, you will have to configure a proper log management policy to prevent log rolling at the time of transfer.
     

FTP clients use curl.


.. admin-log-ftplog:

FTP Log
---------------------

FTP log is merged and saved in /usr/local/ston/sys/stonb/stonb.log. ::

    #Fields: date time local-path cs-url file-size time-taken sc-status sc-error-msg
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10006 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/access_20140423_1700.log ftp://192.168.0.14:21/winesoft.co.kr/access_20140423_1700.log 260 60 success "-"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10008 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/filesystem_20140423_080000.log ftp://192.168.0.14:21/winesoft.co.kr/filesystem_20140423_080000.log 179 60 success "-"

Each field is distinguished by an empty space, and each field specifies the following:

-  ``date`` Date
-  ``time`` Time
-  ``local-path`` Local path of the log to be transferred
-  ``cs-url`` FTP address to transfer the log
-  ``file-size`` File size to be transferred
-  ``time-taken (unit: ms)`` Required time to transfer
-  ``sc-status`` Transfer success/failure(success or fail)
-  ``sc-error-msg`` Curl error message when the transfer fails



.. admin-log-ftptransfer:

Log FTP Transfer
---------------------

When the log is rolling, you can set it to be uploaded with a designated `FTP client`. 
If separated by commas, multiple `FTP clients` can be used at the same time. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>
   
   <Log>
      <Access Ftp="backup1, backup2">ON</Access>
      <Origin Ftp="backup_org">ON</Origin>
      <Monitoring Ftp="backup1">ON</Monitoring>
      <FileSystem Ftp="backup2">ON</FileSystem>   
   </Log>
    
-  ``Ftp`` `FTP client` to use

Log is uploaded to ftp://{FTP server address}/{Virtual host name}/{Rolled log name}. 
For example, the upload address of the rolled log (access_20140424_0000.log) of the virtual host (example.com) in the ftp.dummy.com server will be `ftp://ftp.dummy.com/example.com/access_20140424_0000.log`.
