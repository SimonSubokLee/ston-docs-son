.. _bandwidth_control:

Chapter 14. Bandwidth
******************

This chapter explains various bandwidth control methods for each virtual host.
In the old days, the focus was on limiting bandwidth so it wouldn't exceed a specific level.
Nowadays, on the other hand, effectively controlling bandwidth is more important.
Moreover, you can optimize bandwidth by analyzing content in real time.


.. toctree::
   :maxdepth: 2

Virtual Host Bandwidth Restriction
====================================

A restriction limits the maximum bandwidth of a virtual host.
This physical method has the highest priority. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <TrafficCap Session="0">0</TrafficCap>   
    
-  ``<TrafficCap> (default: 0 Mbps)``
   Configure the maximum bandwidth of a virtual host in Mbps. 
   Setting this value to 0 will not limit bandwidth. 
   The ``Session (default: 0 Kbps)`` property configures the maximum bandwidth of each client session.

For example, if you set ``<TrafficCap>`` to 50 (Mbps), it has the same effect with having 50Mbps NIC.
The sum of bandwidth of all clients that are connected to the relative virtual host cannot exceed 50Mbps. 

The following explains how ``Session`` works:

1. Even if ``Session`` is configured, the sum of bandwidth of all clients cannot exceed the ``<TrafficCap>``.
2. Even if `Bandwidth Throttling`_ is configured, the maximum speed of each client session cannot exceed the ``Session``.


.. _bandwidth-control-bt:

Bandwidth Throttling
====================================

BT(Bandwidth Throttling) dynamically controls client transfer bandwidth for each session.
Media files usually include V(Video) and A(Audio) headers, as seen in the figure below.

.. figure:: img/conf_media_av.png
   :align: center
      
   The header is not a subject of BT.

The header gets bigger when the play-time is longer or the key frame cycle is shorter.
Therefore, if the media file can be recognized, the header has to be transferred without limiting bandwidth for smooth playback.
BT starts after the header transfer is completed, as shown below.

.. figure:: img/conf_bandwidththrottling2.png
   :align: center
      
   Operational Scenario
   
::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Unit="kbps">1000</Bandwidth>
         <Ratio>100</Ratio>
         <Boost>5</Boost>      
      </Settings>
      <Throttling>OFF</Throttling> 
   </BandwidthThrottling>   
    
``<BandwidthThrottling>`` configures the default operation underneath the tag.

-  ``<Settings>``
   
   Configures the default operation.
   
   -  ``<Bandwidth> (default: 1000 Kbps)``   
      configures client transfer bandwidth. 
      ``Unit`` property configures default units ( ``kbps`` , ``mbps`` , ``bytes`` , ``kb`` , ``mb`` ).
   
   -  ``<Ratio> (default: 100 %)``    
      configures the ``<Bandwidth>`` property based on the ratio.
   
   -  ``<Boost> (default: 5 seconds)``   
      transfers data with unlimited speed for a set amount of time.
      The amount of data can be calculated with ``<Boost>`` X ``<Bandwidth>`` X ``<Ratio>``.
         
-  ``<Throttling>``

   -  ``OFF (default)`` Does not apply BT.  
   -  ``ON`` If the conditions list is met, BT is applied.


Bandwidth Throttling Condition List
--------------------------

This section explains how to configure the BT condition list.
BT is applied when the condition list is met.
Check the configuration in sequence to ensure they match the specified conditions.
The transfer policy is saved at /svc/{virtual host name}/throttling.txt. ::

   # /svc/www.example.com/throttling.txt
   # Comma is an identifier, and {condition},{Bandwidth},{Ratio},{Boost} format is used.
   # Other than {condition} field, everything else can be omitted.
   # Omitted field adopts default value from ``<Settings>``.
   # All expressions are identical to acl.txt setting.
   # The unit of {Bandwidth} uses ``Unit`` property of ``<Settings>`` ``<Bandwidth>``.
   
   # After transferring 3 seconds of data to the client with unlimited speed, then limit the transfer speed to 3Mbps(3000Kbps = 2000Kbps X 150%).
   $IP[192.168.1.1], 2000, 150, 3
   
   # This only defines bandwidth. Transfer 5 seconds(default) of data to the client with unlimited speed, then limit the transfer speed to 800 Kbps.
   !HEADER[referer], 800
   
   # This only defines boost. Transfer 10 seconds of data to the client with unlimited speed, then limit the transfer speed to 1000 Kbps.
   HEADER[cookie], , , 10
   
   # This does not apply BT for files with m4a extension.
   $URL[*.m4a], no

If you analyze media files(e.g. MP4, M4A, or MP3), you will get bandwidth at the encoding rate. 
Make sure to use the extension .mp4, .m4a, or .mp3. 
In order to extract bandwidth dynamically, attach **x** to the bandwidth, as shown below. ::

   # If /vod/*.mp4 file is being accessed, find bandwidth. If the bandwidth cannot be found, use 1000 for bandwidth.
   $URL[/vod/*.mp4], 1000x, 120, 5

   # If user-agent header is missing, find bandwidth. If the bandwidth cannot be found, use 500 for bandwidth.
   !HEADER[user-agent], 500x

   # If /low_quality/* file is being accessed, find bandwidth. If the bandwidth cannot be found, use default value for bandwidth.
   $URL[/low_quality/*], x, 200


QueryString Priority Condition
--------------------------

Use a predetermined QueryString to dynamically configure ``<Bandwidth>`` , ``<Ratio>`` , or ``<Boost>``. 
This configuration has priority over BT condition. 

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BandwidthThrottling>
      <Settings>
         <Bandwidth Param="mybandwidth" Unit="mbps">2</Bandwidth>
         <Ratio Param="myratio">100</Ratio>
         <Boost Param="myboost">3</Boost> 
      </Settings>
      <Throttling QueryString="ON">ON</Throttling>
   </BandwidthThrottling>   
    
-  ``<Bandwidth>`` , ``<Ratio>`` , ``Param`` of the ``<Boost>``

    Configures QueryString key based on each purpose.
   
-  ``QueryString`` of the ``<Throttling>``

   - ``OFF (default)`` Does not redefine conditions with the QueryString.
   
   - ``ON`` Redefines conditions with the QueryString.
   
If you configured as in the above example, BT will be dynamically configured.
The configuration below depends on the URL that the client requested. ::

    # Transfer 10 seconds of data with unlimited speed, then the transfer speed will be limited to 1.3Mbps(1mbps X 130%).
    http://www.winesoft.co.kr/video/sample.wmv?myboost=10&mybandwidth=1&myratio=130
    
Not all parameters have to be specified. ::

    http://www.winesoft.co.kr/video/sample.wmv?myratio=150
    
If some conditions are omitted, as in the above example, search for a condition list to define rest conditions(bandwidth and boost, in the above example).
If proper conditions are not found, the default value from ``<Settings>`` will be applied.
Even if a QueryString is specified, when the condition list is set to ``no``(do not apply), BT will not be applied.

When QueryString is being used, you might be confused with :ref:`caching-policy-applyquerystring`. 
If you set :ref:`caching-policy-applyquerystring` as ``ON``, the entire QueryString of a URL requested by client will be recognized except ``BoostParam`` , ``BandwidthParam`` , and ``RatioParam``. ::

   GET /video.mp4?mybandwidth=2000&myratio=130&myboost=10
   GET /video.mp4?tag=3277&myboost=10&date=20130726
        
The above QueryStrings are only used to decide BT and are removed when creating a Caching-Key or sending a request to the origin server. 
Therefore, they are recognized as the following: ::

    GET /video.mp4
    GET /video.mp4?tag=3277&date=20130726
    
    
