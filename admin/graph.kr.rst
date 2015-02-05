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
    
All graphs are provided in 5 different types.

======= =========== =========== =============
Type    Dimension   Time unit   Period
======= =========== =========== =============
dash    205 X 175   5minutes    12hours
day     580 X 203   5minutes    2days (48hours)
week    580 X 203   30minutes   2weeks (14days)
month   580 X 203   2hour       7weeks
year    580 X 203   1day        18months
======= =========== =========== =============

Each graph has at least one graph or at most three graphs. 
Main line is displayed in green and Sub line is displayed in blue. 
Also, from the "Week" graph, Peak line is displayed. 
Peak displays the largest value among smaller units than the current unit.

.. note:
   
   Displaying too many graphs at the same time will consume excessive CPU process and could cause significant quality deterioration. 
   Therefore, you should manage the system to draw one graph at a time.


.. toctree::
   :maxdepth: 2


.. _api-graph-global:

Global Resource
====================================

The global resoure graph only serves for system status or STON related resources. 
In the below table, the asterisk stands for one of 5 types(dash, day, week, month, year).

      
      
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
    
-  ``Main`` Accepted
-  ``Sub`` Closed



Server Socket Usage (Client -> STON)
---------------------
::

    /graph/ssockusage_*.png
    
-  ``Main`` Total(전체??)
-  ``Sub`` Established



Client Socket Event (STON -> Origin server)
---------------------
::

    /graph/csockevent_*.png
    
-  ``Main`` Connected
-  ``Sub`` Closed



Client Socket Usage (STON -> Origin server)
---------------------
::

    /graph/csockusage_*.png
    
-  ``Main`` Total(전체??)
-  ``Sub`` Established



Denied IP Access
---------------------
::

    /graph/acldenied_*.png
    
-  ``Main`` Denied client



Event Que
---------------------
::

    /graph/eq_*.png
    
-  ``Main`` Lengh of the event que



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
    
-  ``Main`` Number of preprocessed URL



TCP Socket
---------------------
::

    /graph/tcpsocket_*.png
    
.. figure:: img/graph_tcpsocket_detail.png


      
.. _api-graph-vhost:

Virtual Host
====================================

The virtual host graph serves for the status of entire or each virtual host. 
You can specify a specific virtual host with vhost parameter, or if the parameter is omitted,
the sum of entire virtual host will be returned. ::

    http://127.0.0.1:10040/graph/vhost/mem_day.png?vhost=example.com
    
In the below table, the asterisk stands for one of 5 types(dash, day, week, month, year).



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



Contents Memory
---------------------
::

    /graph/vhost/mem_*.png
    
-  ``Main`` The amount of contents data loaded on the memory



Delete Pending
---------------------
::

    /graph/vhost/wf2d_*.png
    
-  ``Main`` Number of files in delete pending



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
-  ``Sub`` The client sessions that are being transferred



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
    
-  ``Main`` Number of the origin HTTP responses
-  ``Sub`` Number of the origin HTTP requests



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
-  ``Sub`` Number of the HTTP requests of the origin server 



Origin Server Response Time
---------------------
::

    /graph/vhost/origin_http_res_time1_*.png
    
-  ``Main`` HTTP response time for the request sent to the origin server



Origin Server Completion Time
---------------------
::

    /graph/vhost/origin_http_res_time2_*.png
    
-  ``Main`` HTTP transaction completion time for the request sent to the origin server
