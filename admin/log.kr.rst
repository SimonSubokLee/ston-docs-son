.. admin-log:

Chapter 17. Log
******************

This chapter will explain about the log.
A service begins with the log and ends with the log.
The log is most valuable asset, and a legislation, and a arbitrator of system failure.

There are global and virtual host logs. 
All logs can be turned on or off, and they have identical properties. ::

   <XXX Type="time" Unit="1440" Retention="10" Compression="OFF">ON</XXX>

-  ``Type (default: time)`` and ``Unit (default: 1440 minutes)`` set log rolling conditions.

   - ``time`` Roll the log file for every configured ``unit`` time(unit: minute).
   - ``size`` Roll the log file for every configured ``unit`` size(unit: MB).
   - ``both`` By using a comma(,), time and size can be configured at the same time.
     For example, Unit="1440, 100" configuration rolls the log file for every 24 hours(1440 minutes) or every 100MB.
     
-  ``Retention (default: 10 files)`` Keep the set number of log files.

-  ``Compression (default: OFF)`` Compress the log when rolling.
   For example, when access_20140715_0000.log file is rolling, it is compressed and saved as access_20140715_0000.log.gz.

If the ``Type`` is set to "time" and the ``Unit`` is set to 10, log is rolled for every multiple of 10 minutes.
For example, even if the service started at 2:18, logs will be rolled at 2:20, 2:30, 2:40, and so on. 
Likewise, if you want to roll the log once at midnight, you can set the ``Unit`` to 1440(60 minutes X 24 hours).
In the ``time`` configuration, log will be rolled at least once a day, therefore the maximum value of ``Unit`` cannot exceed 1440.

.. figure:: img/log_rolling1.jpg
   :align: center
   
If you set the maximum value 1440 for Unit, log will be recorded as the below figure.

.. figure:: img/log_rolling2.jpg
   :align: center


.. toctree::
   :maxdepth: 2



.. admin-log-install:

Install Log
====================================

Every details during installation/update will be recorded in the install.log.
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

Info log can be configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>
   
   <InfoLog Type="size" Unit="1" Retention="5">ON</InfoLog>   

-  ``<InfoLog> (default: ON, Type: size, Unit: 1)``   
   Records operation and configuration changes of STON.
   
   
.. _admin-log-deny:

Deny Log
====================================

Deny log is configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>

   <DenyLog Type="size" Unit="1" Retention="5">ON</DenyLog>   

-  ``<DenyLog> (default: ON, Type: size, Unit: 1)``

   Deny log records blocked IP addresses by :ref:`access-control-serviceaccess`. ::
   
      #Fields: date time c-ip deny
      2012.11.15 07:06:10 1.1.1.1 AP
      2012.11.15 07:06:26 2.2.2.2 GIN
      2012.11.15 07:06:30 3.3.3.3 3.3.3.1-255
      
   Each fields are distinguished by an empty space, and each field specifies the following.
   
   - ``date`` date
   - ``time`` time
   - ``c-ip`` client's IP
   - ``deny`` denied condition
   
   
.. _admin-log-originerror:

OriginError Log
====================================

OriginError log is configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>
   
   <OriginErrorLog Type="size" Unit="5" Retention="5" Warning="OFF">ON</OriginErrorLog>   

-  ``<OriginErrorLog> (default: OFF, Type: size, Unit: 5, Warning: OFF)``

   OriginError log records errors only occured in the origin server of all virtual hosts. 
   Errors consist of connection timeouts and receive timeouts, and exclusion/recovery results of the origin server are logged. ::
   
      #Fields: date time vhostname level s-domain s-ip cs-method cs-uri time-taken sc-error sc-resinfo
      2012.11.15 07:06:10 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:26 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:30 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      #2012.11.15 07:06:30 [example.com] 192.168.0.13 excluded from service
      #2012.11.15 07:06:31 [example.com] Origin server list: 192.168.0.14
      #2012.11.15 07:11:11 [example.com] 192.168.0.13 recovered back in service
      #2012.11.15 07:11:12 [example.com] Origin server list: 192.168.0.13
   
   Each fields are distinguished by an empty space, and each field specifies the following.
   
   - ``date`` Date of error occurance
   - ``time`` Time of error occurance
   - ``vhostname`` [Virtual host]
   - ``level`` [Error level(Error or Warning)]
   - ``s-domain`` Origin server domain
   - ``s-ip`` Origin server IP
   - ``cs-method`` HTTP Method sent to the origin server from STON
   - ``cs-uri`` URI sent to the origin server from STON
   - ``time-taken`` Elapsed time to occur an error
   - ``sc-error`` Types of error
   - ``sc-resinfo`` Server response information at the error occurance(distinguished wih a comma)
   
   If ``Warning`` property is set to ``ON``, the following erroneous HTTP communication will be logged. ::
   
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 PartialResponseOnNormalRequest Res=206,Len=2635
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 ClosedWithoutResponse -
      
   An erroneous HTTP communication occurs at the following cases.
   
   - ``ClosedWithoutResponse`` Connection closed by the origin server. HTTP response is not returned.
   - ``ClosedWhenDownloading`` Connection closed by the origin server. Desired Content-Length is not downloaded.
   - ``NotPartialResponseOnRangeRequest`` Range request is not responded with 206 response code.
   - ``DifferentContentLengthOnRangeRequest`` Content-Length is different from requested Range.
   - ``PartialResponseOnNormalRequest`` Response code 206 is returned for non-Range request.



.. admin-log-syslog:

SysLog Transfer
====================================

Use `syslog <http://en.wikipedia.org/wiki/Syslog>`_ protocol to forward logs with UDP in real time. 
All logs can be configured to be transferred to syslog. ::

   # server.xml - <Server><Cache>
   
   <InfoLog SysLog="OFF">ON</InfoLog>
   <DenyLog SysLog="OFF">ON</DenyLog>
   <OriginErrorLog SysLog="OFF">ON</OriginErrorLog>
    
-  ``SysLog``

   - ``OFF (default)`` Does not use syslog.
   
   - ``ON`` Transfer the log to ``<SysLog>`` that is configured underneath of current tag.
   
The following is an example of configuring syslog when ``<OriginErrorLog>`` is being logged. ::

   # server.xml - <Server><Cache>

   <OriginErrorLog SysLog="ON">
      <SysLog Priority="local3.info" Dest="192.168.0.1:514" />
      <SysLog Priority="user.alert" Dest="192.168.0.2" />
      <SysLog Priority="mail.debug" Dest="log.example.com" />
   </OriginErrorLog>
    
1. Set the property of ``SysLog`` in the ``<OriginErrorLog>``.
#. Create the ``<SysLog>`` tag underneath of the ``<OriginErrorLog>``. The log can be transferred to configured number of servers at the same time.
#. Configures the property of ``Priority`` in the ``<SysLog>``. 
   The expression is consist of a combination of `Facility Levels <http://en.wikipedia.org/wiki/Syslog#Facility_levels>`_ in syslog and 
   `Severity levels <http://en.wikipedia.org/wiki/Syslog#Severity_levels>`_.
#. Configures the property of ``Dest`` in the ``<SysLog>``. The ``Dest`` stands for the syslog reception server, and if the reception port is 514, it can be omitted.

The below is a sys log example based on the above configurations. 
the tag in the syslog is logged as STON/{log name}. ::

    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:20 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd 1996 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:22 [ERROR] [example.com] - 192.168.0.14 GET /favicon.ico 1995 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:24 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd22 2020 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] 192.168.0.14:102 excluded from service
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] Origin server list:
    


Saving Log on Each Virtual Host
====================================

Logs of each virtual host are recorded separately. 
Even if the log is set to ``OFF``, :ref:`api-monitoring-logtrace` works as normal. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>

   <Log Dir="/cache_log">
      ... (skip) ...
   </Log>   

-  ``<Log>`` Configure the directory with ``Dir`` attirbute where log will be recorded at. 
   Log is created in the virtual host directory that is underneath of the configured directory.
   


.. _admin-log-dns:

DNS Log
====================================

If the origin server is set as a Domain, records the Resolving result. ::

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

Each fields are distinguished by an empty space, and each field specifies the following.

-  ``date`` Date
-  ``time`` Time
-  ``domain`` Target domain
-  ``ttl`` Valid time of the record(Time To Live)
-  ``ip-list`` IP list
-  ``ip-count`` Number of IP
-  ``time-taken`` Running time
-  ``result`` success or fail


.. _admin-log-access:

Access Log
====================================

Records HTTP transactions of all clients. 
Log is recorded when HTTP transaction is completed, and completed transaction means either transfer completion or transfer interruption. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <Access Type="time" Unit="1440" Retention="10" XFF="on" Form="ston" Local="Off">ON</Access>   
    
-  ``XFF``

   - ``OFF (default)`` Records client IP.
   - ``ON`` Records X-Fowarded-For header value sent from client. If the header value is not exist, this is identical to ``OFF``.

-  ``Form``
   
   - ``ston (default)`` W3C standard + expansion field
   - ``apache`` Apache format
   - ``iis`` IIS format
   - ``custom`` `admin-log-access-custom`

-  ``Local``

   - ``OFF (default)`` Local communications(Loopback) are not logged.
   - ``ON`` Local communications(Loopback) are logged.
  
::

    #Fields: date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) sc-status sc-bytes time-taken cs-referer sc-resinfo cs-range sc-cachehit cs-acceptencoding session-id sc-content-length
    2012.06.27 16:52:24 220.134.10.5 GET /web/h.gif - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 98141 5 - Bypass+gzip+SSL3 - TCP_HIT gzip+deflate 7 1273735
    2012.06.27 16:52:26 220.134.10.5 GET /favicon.ico - 80 - 61.50.7.9 Chrome/19.0.1084.56 200 949 2 - - - TCP_HIT gzip+deflate 35 14875
    2012.06.27 17:00:06 220.168.0.13 GET /setup.Eexe - 80 - 61.168.0.102  Mozilla/5.0+(Windows+NT+6.1;+WOW64)+AppleWebKit/536.11+(KHTML,+like+Gecko)+Chrome/20.0.1132.57+Safari/536.11 206 20971800 7008 - - 398458880-419430399 TCP_HIT - 41 89764358

Each fields are distinguished by an empty space, and each field specifies the following.

-  ``date`` Completed date of HTTP transaction
-  ``time`` Completed time of HTTP transaction
-  ``s-ip`` Server IP
-  ``cs-method`` HTTP Method sent from the client
-  ``cs-uri-stem`` QueryString excluded URL sent from the client
-  ``cs-uri-query`` QueryString of the URL sent from the client
-  ``s-port`` Server port
-  ``cs-username`` Client username
-  ``c-ip`` If the XFF configuration of client IP is set to "ON", records X-Forwarded-For header value.
-  ``cs(User-Agent)`` HTTP User-Agent sent from the client
-  ``sc-status`` Server response code.
-  ``sc-bytes`` Bytes sent from the server(header + contents)
-  ``time-taken`` Total elapsed time until HTTP transaction is completed(millisecond)
-  ``cs-referer`` HTTP Referer sent from the client
-  ``sc-resinfo`` Additional information. Distinguished by "+" character. 
   If encoded contents are serviced, the encoding option(gzip or defalte) is specified. 
   Also security method(SSL3 or TLS1) is specified for a secured communication. 
   "Bypass" will be specified for a bypassed communication.
   
-  ``cs-range`` Logs range header sent from the client.
-  ``sc-cachehit`` Cache HIT result.
-  ``cs-acceptencoding`` Accept-Encoding header sent from the client.
-  ``session-id`` HTTP client session ID (unsigned int64)
-  ``sc-content-length`` Server response Content-Length header value

Access log records all HTTP transactions regardless of success or failure of the transfer. 
HTTP transaction starts when a client sends a HTTP request. 
If HTTP connection is closed before STON sends a response to the client, corresponding HTTP transaction is also considered as completed. 
For both ``sc-status`` and ``sc-bytes`` of the log will be recorded to 0. 
This kind of log is recorded when the client closes the connection before STON receives a response from the origin server.



.. _admin-log-access-custom:

Custom Access Log Format
====================================

Configures to use a customized Access log format. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <Access Form="custom">ON</Access>
   <AccessFormat>%a %A %b id=%{userid}C %f %h %H "%{user-agent}i" %m %P "%r" %s %t %T %X %I %O %R %e %S %K</AccessFormat>   
  
-  ``Form`` attribute of the ``<Access>`` is set to ``custom``.

-  ``<AccessFormat>`` Custom log format.

The following Access log will be recorded for above configurations. (#Fields are not recorded.) ::

    192.168.0.88 192.168.0.12 163276 id=winesoft; image.jpg example.com HTTP "STON" GET 80 "GET /ston/image.jpg?type=png HTTP/1.1" 200 2014-04-03 21:21:54 1 C 204 163276 1 2571978 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 63276 id=winesoft; vod.mp4 example.com HTTP "STON" POST 80 "GET /ston/vod.mp4?start=10 HTTP/1.1" 200 2014-04-03 21:21:54 12 C 304 363276 2 2571979 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 3634276 id=ston; news.html example.com HTTPS "STON" GET 443 "GET /news.html HTTP/1.1" 200 2014-04-03 21:21:54 30 X 156 2632576 1 2571980 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 6332476 id=winesoft; style.css example.com HTTP "STON" HEAD 80 "GET /style.css HTTP/1.1" 200 2014-04-03 21:21:54 10 X 234 653276 2 2571981 TCP_REFRESH_HIT HTTP/1.1
    192.168.0.88 192.168.0.12 6276 id=ston; ui.js example.com HTTP "STON" GET 80 "GET /ui.js HTTP/1.1" 200 2014-04-03 21:21:54 1 X 233 63276 1 2571982 TCP_MISS HTTP/1.1
    192.168.0.88 192.168.0.12 626 id=winesoft; hls.m4u8 example.com HTTP "STON" GET 80 "GET /hls.m4u8 HTTP/1.1" 200 2014-04-03 21:21:54 2 X 124 6312333276 2 2571983 TCP_REFRESH_HIT HTTP/1.1
  
This is developed based on `Apache log format <https://httpd.apache.org/docs/2.2/ko/mod/mod_log_config.html>`_ and there are several expansion fields. 
Specifiers in each field does not have any restrictions, but when you are using a space as a specifier,
you should use double quotation marks for items that could include space such as User-Agent.

-  ``%...a`` Client IP ::

      192.168.0.66
      
-  ``%...A`` Server IP address :: 

      192.168.0.14
      
-  ``%...b`` Size of transferred bytes except the HTTP header ::

      1024
      
-  ``%...{foobar}C`` Contents of Foobar cookie of the request that the server received  ::

      If %{id=}c format is used, then log the id value of the Cookie
      
-  ``%...D`` Elapsed time to process the request(millisecond) ::

      3000
      
-  ``%...f`` File name ::

      If the file name is /mp4/iu.mp4, log iu.mp4
      
-  ``%...h`` HostName ::

      example.com
      
-  ``%...H`` Request protocol ::

      http or https
      
-  ``%...{foobar}i`` Contents of foobar: header of the request that the server received ::

      If %{User-Agent}i format is used, log the User-Agent value
      
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

-  ``%...{format}t`` Date format defined in the Format ::

      If %{%Y-%m-%d %H:%M:%S}T format is used, 2014-08-07 06:12:23 form will be logged.

-  ``%...T`` TimeTaken(second) ::

      10

-  ``%...U`` ShortURI ::

      /img/img.jpg

-  ``%...X`` Status when the transaction is completed
   
   - ``X`` Terminated before the response is completed
   - ``C`` Response is completed
   
   ::
   
      C
    
-  ``%...I`` Received bytes including the request header ::
    
      2048
    
-  ``%...O`` Transmitted bytes including the response header ::
      
      2048
      
-  ``%...R`` Response time(millisecond) ::

      2
      
-  ``%...e`` Session-ID ::

      1
      
-  ``%...S`` Caching HIT result ::

      TCP_HIT
      
-  ``%...K`` Request HTTP version	::

      HTTP/1.1
  
If the configured field value is not existed, then marked as "-". 
If the format is wrong, default format of STON(Form="ston") is adopted.
  
From the above table, you can leave empty spaces("...") in each field(eg. "%h %U %r %b) as blanks or specify record condition (if not satisfied, "-" is logged). 
The condition can be configured with HTTP status code list or NOT condition can be set with an exclamation mark(!). 

The following example only logs User-agent when there is a 400(Bad Request) error or a 501(Not Implemented) error. ::

    "%400,501{User-agent}i" 
  
The following example logs Referers of all abnormal requests. ::
  
    "%!200,304,302{Referer}i"



.. _admin-log-origin:

Origin Log
====================================

This logs all HTTP transaction in the origin server. 
Log is recorded when the HTTP transaction is completed such as at the moment of transfer completion or transfer interruption. ::

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

If there is an origin server failure, an error log starting with #[ERROR:xx] is recorded. 
Each fields are distinguished by an empty space, and each field specifies the following.

.. figure:: img/time_taken.jpg
   :align: center
   
   Time measurement section of the origin

-  ``date`` HTTP transaction completed date
-  ``time`` HTTP transaction completed time
-  ``cs-sid`` Unique ID of the session. HTTP transactions that are processed(recycled) with the same session have the same value.
-  ``cs-tcount`` Transaction count. This value counts how many HTTP transactions are processed in this session. Transactions with an identical ``cs-sid`` cannot have the same value.
-  ``c-ip`` IP of STON
-  ``cs-method`` HTTP Method sent to the origin server
-  ``s-domain`` The origin server domain
-  ``cs-uri`` URI sent to the origin server
-  ``s-ip`` IP of the origin server
-  ``sc-status`` HTTP response code of the origin server
-  ``cs-range`` Range request value sent to the origin server
-  ``sc-sock-error`` Socket error code(1=Transfer timeout, 2=Transfer delay, 3=connection close)
-  ``sc-http-error`` Log the response code when the origin server returns either 4xx or 5xx responses
-  ``sc-content-length`` Content Length sent by the origin server
-  ``cs-requestsize (unit: Bytes)`` The size of HTTP request header sent to the origin server
-  ``sc-responsesize (unit: Bytes)`` The size of HTTP header that the origin server responsed
-  ``sc-bytes (unit: Bytes)`` Received contents size(header excluded)
-  ``time-taken (unit: ms)`` Total elapsed time until the HTTP transaction is completed. If the session is not recycled, socket connection time will be added up
-  ``time-dns (unit: ms)`` Time spent on DNS query
-  ``time-connect (unit: ms)`` Time spent for establishing socket with the origin server
-  ``time-firstbyte (unit: ms)`` Elapsed time from request transmit to response reception
-  ``time-complete (unit: ms)`` Elapsed time from the first response to completion
-  ``cs-reqinfo`` Additional information. Distinguished by "+" character.  "Bypass" will be logged for a bypass communication, and "PrivateBypass" will be logged for private bypass communication.
-  ``cs-acceptencoding`` If compressed contents are requested to the origin server, "gzip+deflate" will be logged.
-  ``sc-cachecontrol`` cache-control header sent from the origin server
-  ``s-port`` The origin server port
-  ``sc-contentencoding`` Content-Encoding header sent from the origin server
-  ``session-id`` The HTTP client session ID(unsigned int64) that caused the origin server request
-  ``session-type`` Session type requested to the origin server

   -  ``cache`` Sessions that are used for caching
   -  ``recovery`` Sessions that are used for recovery in the :ref:`origin_exclusion_and_recovery`
   -  ``healthcheck`` Sessions that are used by :ref:`origin-health-checker`


.. admin-log-monitoring:

Monitoring Log
====================================

Logs average stats of last 5 minutes. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>

   <Monitoring Type="size" Unit="10" Retention="10" Form="json">ON</Monitoring>
  
-  ``Form`` Specifies log format. ( ``json`` or ``xml`` )



.. admin-log-filesystem:

FileSystem Log
====================================

Logs all File I/O transactions generated by the :ref:`filesystem`. ::

   # server.xml - <Server><VHostDefault><Log>
   # vhosts.xml - <Vhosts><Vhost><Log>
   
   <FileSystem Type="time" Unit="1440" Retention="10">ON</FileSystem>
  
FileSystem is logged when the File I/O transaction is completed. 
The transaction completion point is differ by the type of cs-method. ::

    #Fields: date time cs-method cs-path sc-status sc-bytes response-time time-taken sc-cachehit attr session-id
    2012.06.27 16:52:24 ATTR /t 200 0 100 100 TCP_HIT FOLDER 1
    2012.06.27 16:52:24 ATTR /t/2.gif 200 0 100 100 TCP_HIT FILE 1
    2012.06.27 16:52:24 OPEN /file.txt 200 0 100 2000 TCP_HIT FILE 2
    2012.06.27 16:52:24 READ /file.txt 200 1024768 100 2000 TCP_HIT FILE 2
    
-  ``date`` File I/O transaction completed date
-  ``time`` File I/O transaction completed time
-  ``cs-method`` File I/O access type. One of the followings are used.

   -  ``ATTR`` getattr function call. Records to the log when the function is returned
   -  ``OPEN`` File opened without READ. Records to the log when the file is closed
   -  ``READ`` File opened with READ. Records to the log when the file is closed
   
-  ``cs-path`` Access path
-  ``sc-status`` Response code. The followings except the normal service(200) are failure codes.

   -  ``200`` Normal service
   -  ``301`` Bypass required
   -  ``302`` Service denied
   -  ``303`` Redirect required
   -  ``400`` Invalid request
   -  ``401`` Unable to find the virtual host
   -  ``402`` Initialization failure from the origin server
   -  ``500`` Object initialization failure
   -  ``501`` Object Open failure
   -  ``502`` Generating save path failure
   -  ``503`` Memory initialization failure
   -  ``504`` Emergency status
   -  ``600`` Timeout during file service standby
   -  ``601`` Timeout during file data service standby
   -  ``602`` File initialization failure during file service standby
   -  ``603`` Data initialization failure during file data service standby
   -  ``701`` Invalid Offset
   -  ``702`` Specific file section load failure
   -  ``703`` Not enough memory
   -  ``704`` Generating origin session failure
   
-  ``sc-bytes`` Read byte size
-  ``response-time`` Time elapsed from function call to connecting service object
-  ``time-taken`` Time elapsed from function call to completing File I/O Transaction.
-  ``sc-cachehit`` Cache HIT result.
-  ``attr`` FILE or FOLDER
-  ``session-id`` File I/O session ID (unsigned int64)

   .. note::
   
      ``session-id`` is allocated when the Client(HTTP or File I/O) Context is generated. 
      In the general process of Open -> Read -> Close, the Client Context is created at Open and destructed at Close. 
      On the other hand, the getattr function is an atomic funcion that Client Context is created/destructed all the time, so it always assigned with a new session-id.

  
  
.. _admin-log-ftp:

FTP Transfer
====================================

Log is uploaded through designated FTP client when the log is rolling. 


.. _admin-log-ftpclient:

FTP Client
---------------------

Configures FTP client. 
Rolled log is uploaded to the FTP server in real time.

.. figure:: img/conf_ftpclient.png
   :align: center
   
   FTP client architecture and operation

FTP client exists outside of the STON as seen above. 
STON only feeds existing log in the local into the FTP client que without administrating FTP operation. 
FTP client processes upload based on its configuration.

FTP client can be configured in the global setting(server.xml). ::

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

-  ``<Ftp>`` Configures FTP client. The unique name can be configured with the ``Name`` attribute.

   - ``Mode (default: Passive)`` Connection mode ( ``Passive`` or ``Active`` )
   - ``Address`` FTP address. 
   - ``Account`` FTP account. If you want to encrypt the password(eg. qwerty), the below API can be adopted. ::
     
        /command/encryptpassword?plain=qwerty
        
     The encrypted password can be configured as below. ::
     
        <Password Type="enc">dXR9k0xNUZVVYQsK5Bi1cg==</Password>
     
   - ``ConnectTimeout`` Connection pending time
   - ``TransferTimeout`` Transfer pending time
   - ``TrafficCap (unit: KB)`` If this value is greater than 0, it configures the maximum transfer bandwidth.
   - ``DeleteUploaded (default: OFF)`` Deletes corresponding log after transfer completion.
   - ``BackupOnFail (default: OFF)`` Backup corresponding log so that the log is alive after transfer failure. ::
     
        /usr/local/ston/stonb/backup/
     
     Backup log is not retransmitted, and will not be deleted until the administrator deletes it.
   
   - ``UploadPath`` Configures upload path.
     
     - ``%{time format}s`` Log start time
     - ``%{time format}e`` Log end time
     - ``%p`` prefix
     - ``%v`` Virtual host name
     - ``%h`` Device HOST name
     
     If the following configuration is used ::
     
        # server.xml - <Server><Ftp>
        
        <UploadPath>/log_backup/%v/%s-%e.%p.log</UploadPath>
     
     The upload path will be as below. ::
     
        /log_backup/example.com/200140722_0000-200140722_2300.access.log
        
   - ``Transfer`` Determines log transfer time. ``Type`` attribute will determine the format of value.
   
     - ``Rotate (default)`` Transfer the log right after rolling. Does not have a value.
     - ``Static`` Transfer the log once a day at a specific time. For example, this value is set to 04:00, transfer will occur at 4am.
     - ``Interval`` Transfer the log every set interval of time. For example, this value is set to 4, transfer will occur at every 4 hours.
     
     If you want to configure the transfer time, you will have to configure a proper log management policy to prevent log rolling at the time of transfer.
     

FTP client uses curl.


.. admin-log-ftplog:

FTP Log
---------------------

FTP log is merged and saved in /usr/local/ston/sys/stonb/stonb.log. ::

    #Fields: date time local-path cs-url file-size time-taken sc-status sc-error-msg
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10006 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:10:20 /ston_log/winesoft.co.kr/access_20140423_1700.log ftp://192.168.0.14:21/winesoft.co.kr/access_20140423_1700.log 260 60 success "-"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/origin_20140423_080000.log ftp://ftp.winesoft.co.kr:21/winesoft.co.kr/origin_20140423_080000.log 381 10008 fail "curl: (7) couldn't connect to host"
    2014-04-23 17:11:00 /ston_log/winesoft.co.kr/filesystem_20140423_080000.log ftp://192.168.0.14:21/winesoft.co.kr/filesystem_20140423_080000.log 179 60 success "-"

Each fields are distinguished by an empty space, and each field specifies the following.

-  ``date`` Date
-  ``time`` Time
-  ``local-path`` Local path of the log to be transferred
-  ``cs-url`` FTP address to transfer the log
-  ``file-size`` File size to be transferred
-  ``time-taken (unit: ms)`` Required time to transfer
-  ``sc-status`` Transfer success/failure(success or fail)
-  ``sc-error-msg`` curl error message when the transfer fails



.. admin-log-ftptransfer:

Log FTP Transfer
---------------------

When the log is rolling, you can set it to be uploaded with designated `FTP client`. 
If separated with commas, you can use multiple `FTP clients` at the same time. ::

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
For example, upload address of the rolled log(access_20140424_0000.log) of the virtual host(example.com) in the ftp.dummy.com server will be `ftp://ftp.dummy.com/example.com/access_20140424_0000.log`.
