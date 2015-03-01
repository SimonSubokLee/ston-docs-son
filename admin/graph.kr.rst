.. _api-graph:

Appendix B: Graph
******************

All MRTG stats are shown in PNG format graphs. 
Call regulation format requires a unit behind the resource. ::

    # 5 types of CPU graph (dash, day, week, month, year)
    http://127.0.0.1:10040/graph/cpu_dash.png
    http://127.0.0.1:10040/graph/cpu_day.png
    http://127.0.0.1:10040/graph/cpu_week.png
    http://127.0.0.1:10040/graph/cpu_month.png
    http://127.0.0.1:10040/graph/cpu_year.png
    
All graphs are provided in five different types.

======= =========== =========== =============
Type    Dimension   Time unit   Period
======= =========== =========== =============
dash    205 X 175   5minutes    12hours
day     580 X 203   5minutes    2days (48hours)
week    580 X 203   30minutes   2weeks (14days)
month   580 X 203   2hour       7weeks
year    580 X 203   1day        18months
======= =========== =========== =============

A graph has at least one line or at most three lines. 
The main line is displayed in green and the Sub line is displayed in blue. 
Also, in the "Week" graph, a Peak line is displayed. 
The Peak displays the largest value among units smaller than the current unit.

.. note:
   
   Displaying too many graphs at the same time will consume excessive CPU process and could cause significant quality deterioration. 
   Therefore, you should manage the system to draw one graph at a time.


.. toctree::
   :maxdepth: 2


.. _api-graph-global:

Global Resource
====================================

The global resource graph only shows system status or STON related resources. 
In the table below, the asterisk stands for one of five types--dash, day, week, month, or year.

      
      
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
---------------------
::

    /graph/ssockevent_*.png
    
-  ``Main`` Accepted server sockets
-  ``Sub`` Closed server sockets



Server Socket Usage (Client -> STON)
---------------------
::

    /graph/ssockusage_*.png
    
-  ``Main`` Total server sockets
-  ``Sub`` Established server sockets



Client Socket Event (STON -> Origin server)
---------------------
::

    /graph/csockevent_*.png
    
-  ``Main`` Connected client sockets
-  ``Sub`` Closed client sockets



Client Socket Usage (STON -> Origin server)
---------------------
::

    /graph/csockusage_*.png
    
-  ``Main`` Total client sockets
-  ``Sub`` Established client sockets



Denied IP Access
---------------------
::

    /graph/acldenied_*.png
    
-  ``Main`` Denied clients



Event Queue
---------------------
::

    /graph/eq_*.png
    
-  ``Main`` Length of the event queue



Write Pending
---------------------
::

    /graph/wf2w_*.png
    
-  ``Main`` Number of files in write pending


.. _api-graph-urlrewrite:

Successful URL Preprocess
---------------------
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

The virtual host graph shows the status of entire or separate virtual host. 
You can specify a particular virtual host by using vhost parameter, or if the parameter is omitted,
the sum of the entire virtual host will be returned. ::

    http://127.0.0.1:10040/graph/vhost/mem_day.png?vhost=example.com
    
In the table below, the asterisk stands for one of five types--dash, day, week, month, or year.



Hit Ratio
---------------------
::

    /graph/vhost/hitratio_*.png
    
-  ``Main`` Request Hit Ratio
-  ``Sub`` Byte Hit Ratio



Number of Contents
---------------------
::

    /graph/vhost/filecount_*.png
    
.. figure:: img/graph_filecount_detail.png



Content Memory
---------------------
::

    /graph/vhost/mem_*.png
    
-  ``Main`` The amount of content data loaded into memory



Delete Pending
---------------------
::

    /graph/vhost/wf2d_*.png
    
-  ``Main`` Number of files waiting to be deleted



Client Bypass
---------------------
::

    /graph/vhost/client_httpreq_bypass_*.png
    
-  ``Main`` Bypassed client HTTP request



Denied Client Request
---------------------
::

    /graph/vhost/client_httpreq_denied_*.png
    
-  ``Main`` Denied client request



Client Session
---------------------
::

    /graph/vhost/client_http_session_*.png
    
-  ``Main`` The entire client session
-  ``Sub`` Client sessions that are being transferred



Client Traffic
---------------------
::

    /graph/vhost/client_traffic_*.png
    
-  ``Main`` Inbound
-  ``Sub`` Outbound



Client Response
---------------------
::

    /graph/vhost/client_http_res_*.png
    
-  ``Main`` Number of client HTTP responses
-  ``Sub`` Number of client HTTP requests



Client Detail Response
---------------------
::

    /graph/vhost/client_http_res_detail_*.png
    
.. figure:: img/graph_rescode_detail.png



Client Transaction Completion
---------------------
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
---------------------
::

    /graph/vhost/client_http_res_time2_*.png
    
-  ``Main`` HTTP transaction completion time for a client request



Client Caching Response
---------------------
::

    /graph/vhost/client_http_res_hit_*.png
    
.. figure:: img/graph_filehit.png



Client SSL Traffic
---------------------
::

    /graph/vhost/client_traffic_ssl_*.png
    
-  ``Main`` Inbound
-  ``Sub`` Outbound



Origin Server Session
---------------------
::

    /graph/vhost/origin_http_session_*.png
    
-  ``Main`` The entire origin session
-  ``Sub`` The origin sessions that are being transferred



Origin Server Traffic
---------------------
::

    /graph/vhost/origin_traffic_*.png
    
-  ``Main`` Inbound
-  ``Sub`` Outbound



Origin Server Response
---------------------
::

    /graph/vhost/origin_http_res_*.png
    
-  ``Main`` Number of origin HTTP responses
-  ``Sub`` Number of origin HTTP requests



Origin Server Detail Response
---------------------
::

    /graph/vhost/origin_http_res_detail_*.png
    
.. figure:: img/graph_rescode_detail.png



Origin Server Transaction Completion
---------------------
::

    /graph/vhost/origin_http_res_complete_*.png
    
-  ``Main`` Number of completed origin server HTTP responses
-  ``Sub`` Number of HTTP requests made of the origin server 



Origin Server Response Time
---------------------
::

    /graph/vhost/origin_http_res_time1_*.png
    
-  ``Main`` HTTP response time for requests sent to the origin server



Origin Server Completion Time
---------------------
::

    /graph/vhost/origin_http_res_time2_*.png
    
-  ``Main`` HTTP transaction completion time for requests sent to the origin server
