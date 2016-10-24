.. _api-graph:

Appendix A: Graph
******************

All MRTG statistics are shown as PNG-format graphs. The call format is the resource and then the unit. ::

    # 5 types of CPU graphs (dash, day, week, month, year)
    http://127.0.0.1:10040/graph/cpu_dash.png
    http://127.0.0.1:10040/graph/cpu_day.png
    http://127.0.0.1:10040/graph/cpu_week.png
    http://127.0.0.1:10040/graph/cpu_month.png
    http://127.0.0.1:10040/graph/cpu_year.png
    
All graphs are provided in 5 different types.

======= =========== =========== =============
Type    Size        Time unit   Period
======= =========== =========== =============
dash    205 X 175   5 min       12 hours
day     580 X 203   5 min       2 days (48 hours)
week    580 X 203   30 min      2 weeks (14 days)
month   580 X 203   2 hours     7 weeks
year    580 X 203   1 day       18 months
======= =========== =========== =============

A graph can have from one to three lines. The Main line is drawn in green, while the Sub line is drawn in blue. In graphs for "Week" and above, a Peak line is also displayed. The Peak line draws the highest value from smaller units in pink.


.. note::
   
   If too many graphs are made at once, CPU usage will increase sharply and affect the service quality. To prevent this, please make sure to only draw one graph at a time.


.. toctree::
   :maxdepth: 2


.. _api-graph-global:

Global Resource
====================================

The global resource graph shows the system status or resources related to STON. Below, the asterisk can be replaced with one of five types: dash, day, week, month, or year.

      
      
CPU 
---------------------
::

    /graph/cpu_*.png
    
-  ``Main`` Kernel + User
-  ``Sub`` Kernel



STON CPU
---------------------
::

    /graph/stoncpu_*.png
    
-  ``Main`` Kernel + User
-  ``Sub`` Kernel



Memory
---------------------
::

    /graph/mem_*.png
    
-  ``Main`` Total usage
-  ``Sub`` STON usage



IO Wait 
---------------------
::

    /graph/iowait_*.png
    
-  ``Main`` IO Wait



Load Average 
---------------------
::

    /graph/loadavg_*.png
    
-  ``Main`` Load Average



Server Socket Event (Client -> STON)
------------------------------------
::

    /graph/ssockevent_*.png
    
-  ``Main`` Accepted
-  ``Sub`` Closed



Server Socket Usage (Client -> STON)
------------------------------------
::

    /graph/ssockusage_*.png
    
-  ``Main`` Total server sockets
-  ``Sub`` Established server sockets



Client Socket Event (STON -> Origin Server)
-------------------------------------------
::

    /graph/csockevent_*.png
    
-  ``Main`` Connected client sockets
-  ``Sub`` Closed client sockets



Client Socket Usage (STON -> Origin Server)
-------------------------------------------
::

    /graph/csockusage_*.png
    
-  ``Main`` Total client sockets
-  ``Sub`` Established client sockets



Denied IP Accesses
---------------------
::

    /graph/acldenied_*.png
    
-  ``Main`` Denied clients



Event Queue
---------------------
::

    /graph/eq_*.png
    
-  ``Main`` Length of event queue



Write Pending
---------------------
::

    /graph/wf2w_*.png
    
-  ``Main`` Number of write-pending files


.. _api-graph-urlrewrite:

Successful URL Preprocessing
----------------------------
::

    /graph/urlrewrite_*.png
    
-  ``Main`` Number of preprocessed URLs



TCP Socket
---------------------
::

    /graph/tcpsocket_*.png
    
.. figure:: img/graph_tcpsocket_detail.png


      
.. _api-graph-vhost:

Virtual Host
====================================

The virtual host graph shows the status of all hosts or individual hosts. The vhost parameter can be used to choose a specific virtual host, but if it is omitted then it will provide the statistics of all virtual hosts. ::

    http://127.0.0.1:10040/graph/vhost/mem_day.png?vhost=example.com
    
Below, the asterisk can be replaced with one of five types: dash, day, week, month, or year.



Hit Ratio
---------------------
::

    /graph/vhost/hitratio_*.png
    
-  ``Main`` Request Hit Ratio
-  ``Sub`` Byte Hit Ratio



Amount of Content
---------------------
::

    /graph/vhost/filecount_*.png
    
.. figure:: img/graph_filecount_detail.png



Content Memory
---------------------
::

    /graph/vhost/mem_*.png
    
-  ``Main`` Amount of content data loaded into memory



Delete Pending
---------------------
::

    /graph/vhost/wf2d_*.png
    
-  ``Main`` Number of delete-pending files



Client Bypass
---------------------
::

    /graph/vhost/client_httpreq_bypass_*.png
    
-  ``Main`` Bypassed client HTTP requests



Denied Client Requests
----------------------
::

    /graph/vhost/client_httpreq_denied_*.png
    
-  ``Main`` Denied client requests



Client Sessions
---------------------
::

    /graph/vhost/client_http_session_*.png
    
-  ``Main`` Total client sessions
-  ``Sub`` Client sessions in the middle of transfer



Client Traffic
---------------------
::

    /graph/vhost/client_traffic_*.png
    
-  ``Main`` Inbound
-  ``Sub`` Outbound



Client Responses
---------------------
::

    /graph/vhost/client_http_res_*.png
    
-  ``Main`` Number of client HTTP responses
-  ``Sub`` Number of client HTTP requests



Detailed Client Responses
-------------------------
::

    /graph/vhost/client_http_res_detail_*.png
    
.. figure:: img/graph_rescode_detail.png



Client Transaction Completions
------------------------------
::

    /graph/vhost/client_http_res_complete_*.png
    
-  ``Main`` Number of completed client HTTP responses
-  ``Sub`` Number of client HTTP requests



Client Response Time
---------------------
::

    /graph/vhost/client_http_res_time1_*.png
    
-  ``Main`` HTTP response time for a client request



Client Completion Time
----------------------
::

    /graph/vhost/client_http_res_time2_*.png
    
-  ``Main`` HTTP transaction completion time for a client request



Client Caching Response
-----------------------
::

    /graph/vhost/client_http_res_hit_*.png
    
.. figure:: img/graph_filehit.png



Client SSL Traffic
---------------------
::

    /graph/vhost/client_traffic_ssl_*.png
    
-  ``Main`` Inbound
-  ``Sub`` Outbound



Origin Server Sessions
----------------------
::

    /graph/vhost/origin_http_session_*.png
    
-  ``Main`` Total origin sessions
-  ``Sub`` Origin sessions in the middle of transfer



Origin Server Traffic
---------------------
::

    /graph/vhost/origin_traffic_*.png
    
-  ``Main`` Inbound
-  ``Sub`` Outbound



Origin Server Responses
-----------------------
::

    /graph/vhost/origin_http_res_*.png
    
-  ``Main`` Number of origin HTTP responses
-  ``Sub`` Number of origin HTTP requests



Detailed Origin Server Responses
--------------------------------
::

    /graph/vhost/origin_http_res_detail_*.png
    
.. figure:: img/graph_rescode_detail.png



Origin Server Transaction Completions
-------------------------------------
::

    /graph/vhost/origin_http_res_complete_*.png
    
-  ``Main`` Number of completed origin HTTP responses
-  ``Sub`` Number of origin HTTP requests



Origin Server Response Time
---------------------------
::

    /graph/vhost/origin_http_res_time1_*.png
    
-  ``Main`` HTTP response time for requests sent to the origin server



Origin Server Completion Time
-----------------------------
::

    /graph/vhost/origin_http_res_time2_*.png
    
-  ``Main`` HTTP transaction completion time for requests sent to the origin server
